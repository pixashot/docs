# Core Concepts

Understanding the core concepts of Pixashot will help you make the most of its capabilities. This section covers how Pixashot works under the hood, provides an overview of the API, and explains the request flow.

## How Pixashot Works

Pixashot uses advanced browser automation technology to capture high-quality screenshots of web pages. Here's a simplified explanation of how it works:

1. **Browser Initialization**: Pixashot maintains a pool of headless browser instances using Playwright, a powerful browser automation library.

2. **Page Loading**: When a screenshot request is received, Pixashot loads the specified URL in one of the available browser instances.

3. **Rendering**: The page is fully rendered, including dynamic content and JavaScript-based elements.

4. **Customization**: Any specified customizations (like viewport size, custom scripts, or CSS) are applied.

5. **Capture**: The screenshot is captured according to the specified parameters (full page, element selection, etc.).

6. **Processing**: The captured image is processed (e.g., converted to the requested format, compressed if needed).

7. **Delivery**: The final screenshot is delivered to the client as a response to the API request.

This process happens rapidly, usually within a few seconds, depending on the complexity of the web page and the requested customizations.

## API Overview

Pixashot provides a RESTful API that allows you to easily integrate screenshot capabilities into your applications. The API consists of a single endpoint with various parameters to customize the screenshot capture process.

### Main Endpoint

```
POST /capture
```

### Key Parameters

- `url`: The URL of the web page to capture
- `format`: Output format (png, jpeg, webp, pdf)
- `full_page`: Whether to capture the full scrollable page
- `viewport_width`: Width of the viewport
- `viewport_height`: Height of the viewport
- `delay`: Delay before capture (in milliseconds)
- `selector`: CSS selector to capture a specific element
- `custom_js`: Custom JavaScript to execute before capture

### Example Request

```json
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "viewport_width": 1920,
  "viewport_height": 1080,
  "delay": 1000,
  "custom_js": "document.body.style.backgroundColor = 'red';"
}
```

### Response

The API responds with the captured screenshot in the requested format, along with appropriate headers.

## Request Flow

Understanding the flow of a Pixashot request can help you optimize your usage and troubleshoot any issues. Here's a step-by-step breakdown of what happens when you make a request to Pixashot:

1. **API Request**: Your application sends a POST request to the `/capture` endpoint with the necessary parameters.

2. **Authentication**: Pixashot verifies the provided authentication token or signed URL.

3. **Parameter Validation**: The API validates the request parameters, ensuring all required fields are present and have valid values.

4. **Browser Assignment**: Pixashot assigns an available browser instance from its pool to handle the request.

5. **Page Navigation**: The browser navigates to the specified URL and waits for the page to load.

6. **Custom Scripting**: If provided, custom JavaScript is injected and executed on the page.

7. **Delay**: If a delay is specified, Pixashot waits for the specified duration.

8. **Screenshot Capture**: The screenshot is captured according to the specified parameters (full page, element selection, etc.).

9. **Image Processing**: The captured screenshot is processed into the requested format.

10. **Response**: The final screenshot is sent back to the client as the API response.

11. **Cleanup**: The browser instance is reset and returned to the pool for future requests.

This entire process typically occurs within a few seconds, depending on the complexity of the web page and the specified parameters.

Understanding these core concepts will help you use Pixashot more effectively and troubleshoot any issues that may arise. In the next sections, we'll dive deeper into specific features and advanced usage scenarios.