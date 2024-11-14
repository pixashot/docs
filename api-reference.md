---
excerpt: "Comprehensive API reference for Pixashot, detailing endpoints, request parameters, response formats, error handling, and rate limiting."
published_at: true
---

# API Reference

This section provides a comprehensive reference for the Pixashot API, including details on endpoints, request parameters, response formats, error handling, and rate limiting.

## Endpoints

Pixashot provides a single main endpoint for capturing screenshots:

```
POST https://your-pixashot-instance.com/capture
```

This endpoint accepts various parameters to customize the screenshot capture process.

## Request Parameters

The `/capture` endpoint accepts the following parameters in the request body as JSON:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes* | The URL of the webpage to capture. |
| `html_content` | string | Yes* | HTML content to render and capture instead of a URL. |
| `format` | string | No | Output format: "png" (default), "jpeg", "webp", "pdf", or "html". |
| `full_page` | boolean | No | Capture the full scrollable page. Default is `false`. |
| `window_width` | integer | No | Width of the browser viewport in pixels. Default is 1920. |
| `window_height` | integer | No | Height of the browser viewport in pixels. Default is 1080. |
| `selector` | string | No | CSS selector of a specific element to capture. |
| `delay_capture` | integer | No | Delay in milliseconds before taking the screenshot. Default is 0. |
| `wait_for_timeout` | integer | No | Timeout in milliseconds to wait for page load. Default is 8000. |
| `wait_for_selector` | string | No | Wait for a specific selector to appear in DOM before capturing. |
| `wait_for_network` | string | No | Specify whether to wait for network to be "idle" (default) or "mostly_idle". |
| `custom_js` | string | No | Custom JavaScript to inject and execute before capturing. |
| `image_quality` | integer | No | Image quality (0-100) for JPEG and WebP formats. Default is 90. |
| `pixel_density` | float | No | Device scale factor (DPR) for the screenshot. Default is 1.0. |
| `omit_background` | boolean | No | Render a transparent background. Default is `false`. |
| `dark_mode` | boolean | No | Enable dark mode for the screenshot. Default is `false`. |
| `block_media` | boolean | No | Block images, video, and audio from loading. Default is `false`. |
| `geolocation` | object | No | Geolocation to spoof: `{latitude: float, longitude: float, accuracy: float}`. |

*Note: Either `url` or `html_content` must be provided, but not both.

### ⚠️ Environment-Controlled Features

The following features are controlled by environment variables at the server level and cannot be modified per request:

- Popup blocking (controlled by `USE_POPUP_BLOCKER` environment variable)
- Cookie consent blocking (controlled by `USE_COOKIE_BLOCKER` environment variable)
- Rate limiting (controlled by `RATE_LIMIT_*` environment variables)

Contact your Pixashot administrator to modify these settings.

### Example Request

```json
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "window_width": 1280,
  "window_height": 800,
  "wait_for_network": "idle",
  "wait_for_timeout": 8000,
  "custom_js": "document.body.style.backgroundColor = 'lightblue';"
}
```

## Response Format

The response format depends on the `format` parameter specified in the request:

- For image formats (png, jpeg, webp), the response will be the binary image data with the appropriate `Content-Type` header.
- For PDF format, the response will be the binary PDF data with `Content-Type: application/pdf`.
- For HTML format, the response will be the captured HTML content with `Content-Type: text/html`.

The response will include the following headers:

- `Content-Type`: The MIME type of the response
- `Content-Disposition: attachment; filename=screenshot.<format>`

## Error Handling

Pixashot uses standard HTTP status codes to indicate the success or failure of an API request. In case of an error, the response body will contain a JSON object with more details about the error.

Common status codes:

- 200 OK: The request was successful.
- 400 Bad Request: The request was invalid or cannot be served.
- 401 Unauthorized: The request lacks valid authentication credentials.
- 403 Forbidden: The server understood the request but refuses to authorize it.
- 429 Too Many Requests: Rate limit exceeded (if enabled).
- 500 Internal Server Error: The server encountered an unexpected condition.
- 504 Gateway Timeout: The request timed out.

Error response body format:

```json
{
  "status": "error",
  "message": "Detailed error message",
  "errorType": "ErrorClassName",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

## Rate Limiting

Rate limiting is configured through environment variables on the server:

- `RATE_LIMIT_ENABLED`: Enables/disables rate limiting
- `RATE_LIMIT_CAPTURE`: Defines the rate limit for the capture endpoint
- `RATE_LIMIT_SIGNED`: Defines the rate limit for signed URL requests

When rate limiting is enabled and you exceed the limits, you'll receive a 429 Too Many Requests response:

```json
{
  "status": "error",
  "message": "Rate limit exceeded. Please try again later.",
  "errorType": "RateLimitExceededError",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

## Browser Context

Pixashot uses a single browser context for all requests, which means:

1. Better performance and resource utilization
2. Consistent behavior across requests
3. Extension settings (popup/cookie blocking) apply to all requests
4. Session data is not shared between requests for security

## Best Practices

1. **Timeouts**:
    - Set appropriate `wait_for_timeout` values (default: 8000ms)
    - Use `wait_for_network` strategically ("idle" or "mostly_idle")

2. **Resource Optimization**:
    - Use `block_media: true` when images aren't needed
    - Set appropriate viewport sizes to minimize memory usage

3. **Error Handling**:
    - Implement exponential backoff for retry logic
    - Handle 429 responses properly when rate limiting is enabled

4. **Security**:
    - Keep your AUTH_TOKEN secure
    - Validate and sanitize all URLs before sending
    - Use HTTPS for all API requests

Remember to handle errors gracefully and implement appropriate retry logic in your applications. For high-volume usage, contact your Pixashot administrator to discuss rate limit adjustments and resource allocation.