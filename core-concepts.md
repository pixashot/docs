---
excerpt: "Overview of Pixashot's core concepts, explaining how it works, API structure, and the request flow process."
published_at: true
---

# Core Concepts

Understanding the core concepts of Pixashot will help you make the most of its capabilities. This section covers how Pixashot works under the hood, provides an overview of the API, and explains the request flow.

## How Pixashot Works

Pixashot uses advanced browser automation technology to capture high-quality screenshots of web pages. Here's a detailed explanation of its architecture and operation:

### Single-Context Architecture

Pixashot uses a single browser context architecture for optimal performance and resource utilization:

1. **Browser Initialization**:
    - One Playwright browser instance is launched during startup
    - Browser extensions (popup blocker, cookie consent blocker) are configured via environment variables
    - A single browser context is created and maintained throughout the service lifetime

2. **Worker Processes**:
    - Multiple worker processes handle incoming requests (configurable via `WORKERS`)
    - Each worker shares the same browser context
    - Workers are recycled after handling a configured number of requests (`MAX_REQUESTS`)

### Screenshot Process

When a request is received:

1. **Page Creation**: A new page is created within the shared browser context.

2. **Page Configuration**:
    - Viewport size is set
    - Network conditions are configured
    - Custom headers are applied if specified

3. **Resource Management**:
    - Media blocking is applied if requested
    - Network idle detection is configured
    - Timeout values are set

4. **Page Loading**:
    - The URL is loaded or HTML content is rendered
    - Network conditions are monitored
    - Dynamic content is handled

5. **Page Customization**:
    - Custom JavaScript execution
    - Dark mode application (if requested)
    - Geolocation spoofing (if specified)

6. **Screenshot Capture**:
    - Full-page or element-specific capture
    - Format and quality settings applied
    - Image processing and optimization

7. **Cleanup**:
    - Page is closed
    - Resources are released
    - Context is maintained for future requests

## API Overview

Pixashot provides a RESTful API with a single endpoint and comprehensive customization options.

### Main Endpoint

```
POST /capture
```

### Key Parameters

- `url` or `html_content`: Source content to capture
- `format`: Output format (png, jpeg, webp, pdf)
- `full_page`: Whether to capture the full scrollable page
- `window_width`: Width of the viewport
- `window_height`: Height of the viewport
- `wait_for_network`: Network idle strategy ("idle" or "mostly_idle")
- `wait_for_timeout`: Maximum wait time (milliseconds)
- `custom_js`: Custom JavaScript to execute
- `selector`: CSS selector for element capture

### Example Request

```json
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "window_width": 1280,
  "window_height": 720,
  "wait_for_network": "idle",
  "wait_for_timeout": 8000,
  "block_media": true
}
```

### Environment Configuration

Server-wide settings configured through environment variables:

- `USE_POPUP_BLOCKER`: Enable/disable popup blocking
- `USE_COOKIE_BLOCKER`: Enable/disable cookie consent blocking
- `RATE_LIMIT_ENABLED`: Enable/disable rate limiting
- `CACHE_MAX_SIZE`: Enable/disable response caching

## Request Flow

Here's the complete flow of a Pixashot request:

1. **Request Receipt**:
    - Worker process accepts the request
    - Request parameters are validated
    - Authentication is verified (token or signed URL)

2. **Page Preparation**:
    - New page created in shared context
    - Viewport and settings configured
    - Extensions and blockers already active from context

3. **Content Loading**:
   ```mermaid
   graph TD
     A[Request Received] --> B[Create Page]
     B --> C[Configure Settings]
     C --> D[Load Content]
     D --> E[Wait for Network]
     E --> F[Execute Custom JS]
     F --> G[Capture Screenshot]
     G --> H[Process Image]
     H --> I[Send Response]
   ```

4. **Resource Management**:
    - Browser context remains active
    - Page is properly closed
    - Memory is managed through worker recycling

5. **Response Handling**:
    - Image processing and optimization
    - Format conversion if needed
    - Response streaming for large captures
    - Error handling and reporting

### Performance Considerations

The single-context architecture provides several benefits:

1. **Resource Efficiency**:
    - Shared browser resources
    - Reduced memory overhead
    - Faster page creation

2. **Consistent Behavior**:
    - Extensions work consistently
    - Settings persist across requests
    - Predictable performance

3. **Scalability**:
    - Worker processes handle concurrency
    - Context sharing reduces overhead
    - Efficient resource utilization

4. **Limitations**:
    - Extension settings are server-wide
    - Context sharing requires careful resource management
    - Worker recycling needed for memory management

## Best Practices

1. **Resource Optimization**:
    - Use appropriate `wait_for_network` settings
    - Enable `block_media` when images aren't needed
    - Set realistic viewport sizes

2. **Error Handling**:
    - Implement retry logic with backoff
    - Monitor request timeouts
    - Check health endpoints regularly

3. **Performance Tuning**:
    - Configure worker count based on CPU cores
    - Set appropriate request limits
    - Use caching when possible

4. **Security**:
    - Use HTTPS for all requests
    - Implement rate limiting
    - Keep authentication tokens secure

Understanding these core concepts helps you optimize your use of Pixashot and troubleshoot any issues that arise. For more detailed information about specific features, see our [Advanced Usage](advanced-usage.md) and [API Reference](api-reference.md) documentation.