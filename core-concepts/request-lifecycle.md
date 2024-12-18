---
title: Request Lifecycle
excerpt: Detailed examination of how Pixashot processes screenshot requests, from initial validation through to response delivery.
meta:
  nav_order: 32
---

# Request Lifecycle

This guide details how Pixashot processes screenshot requests, explaining each stage from initial receipt to final response. Understanding this lifecycle is crucial for debugging, optimization, integration development, and contributing to the project.

## Overview

The request lifecycle consists of several distinct phases:

```mermaid
sequenceDiagram
    participant C as Client
    participant V as Request Validation
    participant A as Authentication
    participant B as Browser Context
    participant P as Page Processing
    participant S as Screenshot Capture
    participant R as Response Handler

    C->>V: HTTP Request
    Note over V: Parameter Validation using<br>CaptureRequest model
    V->>A: Authentication Check
    A-->>V: Authentication Result
    V->>B: Request Page from Context
    B->>P: Create & Configure Page
    P->>P: Load Content (URL or HTML)
    P->>P: Execute Interactions (if any)
    P->>S: Capture Screenshot
    S->>R: Format Response (e.g., to binary, JSON)
    R->>C: HTTP Response (with data or error)
````

## 1\. Request Reception and Validation

### Initial Validation

Requests are received and validated against the `CaptureRequest` model defined in [`capture_request.py`](https://github.com/pixashot/pixashot/blob/develop/src/capture_request.py). This model uses Pydantic to ensure type safety and data integrity:

```python
class CaptureRequest(BaseModel):
    url: Optional[HttpUrl] = Field(None, description="URL to capture")
    html_content: Optional[str] = Field(None, description="HTML content to render")
    format: Literal["png", "jpeg", "webp", "pdf", "html"] = "png"
    full_page: bool = False
    window_width: int = 1920
    window_height: int = 1080
    # ... additional fields ...

    @model_validator(mode='before')
    def validate_url_or_html_content(cls, values):
        if not values.get('url') and not values.get('html_content'):
            raise ValueError('Either url or html_content must be provided')
        if values.get('url') and values.get('html_content'):
            raise ValueError('Cannot provide both url and html_content')
        return values
```

### Authentication Check

Authentication is performed using the `verify_auth_token` function defined in [`request_auth.py`](https://github.com/pixashot/pixashot/blob/develop/src/request_auth.py). It checks for a valid `Authorization` header or verifies signed URLs:

```python
def verify_auth_token(auth_header):
    if not config.AUTH_TOKEN:
        return True
    
    token = auth_header.split(' ')[1] if auth_header and len(auth_header.split(' ')) > 1 else None
    return token == config.AUTH_TOKEN
```

### Template Application

If a `template` parameter is provided in the request, it's applied using the `apply_template` validator in the `CaptureRequest` model. Templates are defined in `templates.json` and loaded by `src/templates.py`:

```python
@model_validator(mode='before')
def apply_template(cls, values):
    template_name = values.get('template')
    if template_name:
        template = get_template(template_name)
        if template:
            for key, value in template.items():
                if key not in values or values[key] is None:
                    values[key] = value
    return values
```

## 2\. Browser Context and Page Handling

### Page Creation

A new page is created within the shared browser context. The `ContextManager` (defined in [`context_manager.py`](https://github.com/pixashot/pixashot/blob/develop/src/context_manager.py)) handles the creation and management of the browser context. The `CaptureService` requests a new page:

```python
async def capture_screenshot(self, output_path, options):
    try:
        page = await self.context.new_page()
        # ... rest of the capture logic ...
    except Exception as e:
        raise ScreenshotServiceException(str(e))
```

### Page Configuration

The page is configured according to the request parameters. This includes setting the user agent, geolocation, and other page settings. This logic is primarily handled by `MainBrowserController`.

```python
async def _configure_page(self, page: Page, options):
    """Configure page with user agent and other settings."""
    if getattr(options, 'use_random_user_agent', True):
        headers = self.context_manager._generate_headers(options)
        await page.set_extra_http_headers(headers)

    if options.geolocation:
        await self.set_geolocation(page, options.geolocation)
```

## 3\. Content Loading

### URL Navigation

For URL-based requests, Pixashot navigates to the specified URL with a timeout and retry mechanism:

```python
async def _resilient_navigation(self, page: Page, url: str, timeout: int):
    try:
        await page.goto(
            str(url),
            wait_until='domcontentloaded',
            timeout=timeout
        )
    except Exception as nav_error:
        logger.warning(f"Navigation timeout or error: {str(nav_error)}. Continuing...")
        await page.wait_for_timeout(1000)  # Additional wait time
```

### Content Preparation

The `MainBrowserController` class handles various page preparation steps, such as waiting for the network, applying dark mode, and executing custom JavaScript:

```python
async def prepare_page(self, page: Page, options):
    await self.prevent_horizontal_overflow(page)
    
    if options.wait_for_network in ('idle', 'mostly_idle'):
        timeout = min(options.wait_for_timeout, 5000)
        await self.interaction_controller.wait_for_network_idle(page, timeout)

    if options.dark_mode:
        await self.apply_dark_mode(page)

    if options.wait_for_animation:
        await self.interaction_controller.wait_for_animations(page)
```

## 4\. Interaction Execution

### Page Interactions

The `InteractionController` handles user interactions defined in the request. Supported actions include clicking, typing, hovering, scrolling, and waiting:

```python
async def perform_interactions(self, page: Page, interactions: list):
    for step in interactions:
        try:
            if step.action == "click":
                await self._click(page, step.selector)
            # ... other interaction handling ...
        except Exception as e:
            raise InteractionException(f"Failed to perform {step.action}: {str(e)}")
```

### Wait Strategies

Different wait strategies are implemented to ensure that dynamic content is fully loaded before capture.

* **Network Idle:** `wait_for_network_idle`
* **Mostly Idle:** `wait_for_network_mostly_idle` (defined in `InteractionController`)
* **Selector:** `wait_for_selector`
* **Timeout:** `wait_for_timeout`

## 5\. Screenshot Capture

### Capture Preparation

The `ScreenshotController` prepares the page for capture based on the requested options. It handles both full-page and viewport screenshots:

```python
async def prepare_for_full_page_screenshot(self, page: Page, window_width: int):
    # ... logic to prepare for full page capture ...

async def prepare_for_viewport_screenshot(self, page: Page, window_width: int, window_height: int):
    # ... logic to prepare for viewport capture ...
```

### Screenshot Taking

The actual screenshot is taken using Playwright's `page.screenshot()` method. Fallback mechanisms are implemented to handle potential timeouts or errors:

```python
async def take_screenshot(self, page: Page, options: dict) -> bytes:
    # ... screenshot options ...
    
    try:
        return await self._take_screenshot_with_retry(page, screenshot_options)
    except TimeoutError as e:
        logger.warning(f"Screenshot timeout: {str(e)}. Attempting fallback...")
        return await self._take_fallback_screenshot(page, screenshot_options)
```

## 6\. Response Processing

### Format Handling

The captured screenshot is processed and returned in the requested format. Supported formats include PNG, JPEG, WebP, PDF, and HTML. The response can be formatted as raw binary data, base64 encoded, or an empty response with just a status code.

```python
if options.response_type == 'empty':
    return '', 204
elif options.response_type == 'json':
    return jsonify({
        'file': base64.b64encode(file_data).decode('utf-8'),
        'format': options.format
    }), 200
else:  # by_format
    mime_type = 'application/pdf' if options.format == 'pdf' else f'image/{options.format}'
    response = await make_response(file_data)
    response.headers['Content-Type'] = mime_type
    response.headers['Content-Disposition'] = f'attachment; filename=screenshot.{options.format}'
    return response
```

### Resource Cleanup

After processing the request, resources such as temporary files and the page object are cleaned up.

```python
try:
    # Capture screenshot
    await capture_service.capture_screenshot(output_path, options)
finally:
    # Ensure cleanup
    if os.path.exists(output_path):
        os.remove(output_path)
```

## Error Handling

Errors are caught and handled at each stage of the request lifecycle. Detailed error information is logged and, optionally, returned in the response, especially in development mode:

```python
@app.errorhandler(Exception)
def handle_error(error):
    # ... error handling logic ...
```

## Monitoring and Debugging

### Request Tracking

Each request can be tracked through its lifecycle using logging statements. Key metrics and events are logged for debugging and monitoring.

### Performance Metrics

Performance metrics such as request duration, memory usage, and network activity are collected and can be monitored through the `/health` endpoint or integrated with external monitoring systems.

## Next Steps

* Learn about [Resource Management](resource-management.md) for details on how Pixashot manages memory, CPU, and browser resources.
* Explore [Advanced Features](../api-reference/request-options.md) to understand the full range of capture options.
* Understand [Error Handling](../troubleshooting/debugging.md) for troubleshooting and debugging techniques.
* Deep Dive into [Browser Context Details](browser-context.md) for greater understanding of how the browser context is managed.

Understanding the request lifecycle is crucial for:

* **Debugging issues**: Identifying bottlenecks or failures in the capture process.
* **Optimizing performance**: Fine-tuning capture options and resource usage.
* **Implementing integrations**: Building custom integrations and client libraries.
* **Monitoring and logging**: Tracking key metrics for operational insight.
* **Contributing to the project**: Understanding the flow for extending and improving Pixashot.