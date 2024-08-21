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
| `wait_for_timeout` | integer | No | Timeout in milliseconds to wait for page load. Default is 30000. |
| `wait_for_selector` | string | No | Wait for a specific selector to appear in DOM before capturing. |
| `custom_js` | string | No | Custom JavaScript to inject and execute before capturing. |
| `image_quality` | integer | No | Image quality (0-100) for JPEG and WebP formats. Default is 90. |
| `pixel_density` | float | No | Device scale factor (DPR) for the screenshot. Default is 1.0. |
| `omit_background` | boolean | No | Render a transparent background. Default is `false`. |
| `dark_mode` | boolean | No | Enable dark mode for the screenshot. Default is `false`. |
| `use_popup_blocker` | boolean | No | Use the built-in popup blocker. Default is `true`. |
| `use_cookie_blocker` | boolean | No | Use the built-in cookie consent blocker. Default is `true`. |
| `block_media` | boolean | No | Block images, video, and audio from loading. Default is `false`. |
| `geolocation` | object | No | Geolocation to spoof: `{latitude: float, longitude: float, accuracy: float}`. |

*Note: Either `url` or `html_content` must be provided, but not both.

### Example Request

```json
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "window_width": 1280,
  "window_height": 800,
  "delay_capture": 1000,
  "custom_js": "document.body.style.backgroundColor = 'lightblue';"
}
```

## Response Format

The response format depends on the `format` parameter specified in the request:

- For image formats (png, jpeg, webp), the response will be the binary image data with the appropriate `Content-Type` header.
- For PDF format, the response will be the binary PDF data with `Content-Type: application/pdf`.
- For HTML format, the response will be the captured HTML content with `Content-Type: text/html`.

The response will also include the following headers:

- `Content-Disposition: attachment; filename=screenshot.<format>`
- `X-Pixashot-Capture-Time: <capture_time_in_ms>`

## Error Handling

Pixashot uses standard HTTP status codes to indicate the success or failure of an API request. In case of an error, the response body will contain a JSON object with more details about the error.

Common status codes:

- 200 OK: The request was successful.
- 400 Bad Request: The request was invalid or cannot be served.
- 401 Unauthorized: The request lacks valid authentication credentials.
- 403 Forbidden: The server understood the request but refuses to authorize it.
- 429 Too Many Requests: The user has sent too many requests in a given amount of time.
- 500 Internal Server Error: The server encountered an unexpected condition that prevented it from fulfilling the request.

Error response body format:

```json
{
  "status": "error",
  "message": "Detailed error message",
  "errorType": "ErrorClassName"
}
```

## Rate Limiting

To ensure fair usage and maintain service quality, Pixashot implements rate limiting. The current rate limits are:

- 60 requests per minute per API key
- 1000 requests per hour per API key

If you exceed these limits, you'll receive a 429 Too Many Requests response. The response will include the following headers:

- `X-RateLimit-Limit`: The number of requests allowed in the current period
- `X-RateLimit-Remaining`: The number of requests remaining in the current period
- `X-RateLimit-Reset`: The time at which the current rate limit window resets, in UTC epoch seconds

When you receive a rate limit error, you should wait until the rate limit window resets before making further requests.

### Example Rate Limit Error Response

```json
{
  "status": "error",
  "message": "Rate limit exceeded. Please try again in 37 seconds.",
  "errorType": "RateLimitExceededError"
}
```

This comprehensive API reference should help you integrate Pixashot into your applications effectively. Remember to handle errors gracefully and respect the rate limits to ensure smooth operation of your screenshot capture processes.