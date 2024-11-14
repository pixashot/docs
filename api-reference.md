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

### Basic Options
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string (HttpUrl) | Yes* | URL of the site to take a screenshot of |
| `html_content` | string | Yes* | HTML content to render and capture |
| `template` | string | No | Name of the optional template to use |

*Note: Either `url` or `html_content` must be provided, but not both.

### Viewport and Screenshot Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `window_width` | integer | 1920 | Width of the browser viewport in pixels |
| `window_height` | integer | 1080 | Height of the browser viewport in pixels |
| `full_page` | boolean | false | Take a screenshot of the full page |
| `selector` | string | null | CSS selector of a specific element to capture |

### User Agent Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `use_random_user_agent` | boolean | false | Use a random user agent |
| `user_agent_device` | string | null | Device type: "desktop" or "mobile" |
| `user_agent_platform` | string | null | Platform: "windows", "macos", "ios", "linux", "android" |
| `user_agent_browser` | string | "chrome" | Browser: "chrome", "edge", "firefox", "safari" |

### Format and Response Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `format` | string | "png" | Output format: "png", "jpeg", "webp", "pdf", "html" |
| `response_type` | string | "by_format" | Response type: "by_format", "empty", "json" |

### Interaction Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `interactions` | array | null | List of interaction steps to perform before capturing |
| `wait_for_animation` | boolean | false | Wait for animations to complete before capturing |

### Image Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `image_quality` | integer (0-100) | 90 | Image quality for JPEG and WebP formats |
| `pixel_density` | float | 1.0 | Device scale factor (DPR) |
| `omit_background` | boolean | false | Render a transparent background |
| `dark_mode` | boolean | false | Enable dark mode for the screenshot |

### Wait and Timeout Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `wait_for_timeout` | integer | 8000 | Timeout in milliseconds to wait for page load |
| `wait_for_selector` | string | null | Wait for a specific selector to appear in DOM |
| `delay_capture` | integer | 0 | Delay in milliseconds before taking the screenshot |
| `wait_for_network` | string | "idle" | Network wait strategy: "idle" or "mostly_idle" |

### Network and Resource Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `custom_js` | string | null | Custom JavaScript to inject and execute |
| `ignore_https_errors` | boolean | true | Ignore HTTPS errors during navigation |
| `block_media` | boolean | false | Block images, video, and audio from loading |
| `custom_headers` | object | null | Custom headers to be sent with the request |

### Geolocation Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `geolocation` | object | null | Geolocation to spoof: `{latitude: float, longitude: float, accuracy: float}` |

### PDF-specific Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `pdf_print_background` | boolean | true | Print background graphics in PDF |
| `pdf_scale` | float | 1.0 | Scale of the webpage rendering |
| `pdf_page_ranges` | string | null | Paper ranges to print, e.g., '1-5, 8, 11-13' |
| `pdf_format` | string | "A4" | Paper format: "A4", "Letter", "Legal" |
| `pdf_width` | string | null | Paper width, accepts values labeled with units |
| `pdf_height` | string | null | Paper height, accepts values labeled with units |

### Interaction Step Structure
When using the `interactions` parameter, each step should have the following structure:

```json
{
  "action": "click" | "type" | "hover" | "scroll" | "wait_for",
  "selector": "string",  // Required for click, type, hover
  "text": "string",      // Required for type
  "x": "integer",        // Required for scroll
  "y": "integer",        // Required for scroll
  "wait_for": {          // Required for wait_for action
    "type": "network_idle" | "network_mostly_idle" | "selector" | "timeout",
    "value": "string" | "integer"  // String for selector, integer for others
  }
}
```

### ⚠️ Environment-Controlled Features

The following features are controlled by environment variables at the server level and cannot be modified per request:

- Popup blocking (controlled by `USE_POPUP_BLOCKER` environment variable)
- Cookie consent blocking (controlled by `USE_COOKIE_BLOCKER` environment variable)
- Rate limiting (controlled by `RATE_LIMIT_*` environment variables)

Contact your Pixashot administrator to modify these settings.

### Example Request

Basic screenshot:
```json
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "window_width": 1280,
  "window_height": 800
}
```

Advanced screenshot with interactions:
```json
{
  "url": "https://example.com",
  "format": "png",
  "wait_for_network": "idle",
  "wait_for_timeout": 8000,
  "pixel_density": 2.0,
  "dark_mode": true,
  "interactions": [
    {
      "action": "click",
      "selector": "#accept-cookies"
    },
    {
      "action": "wait_for",
      "wait_for": {
        "type": "network_idle",
        "value": 2000
      }
    }
  ],
  "custom_js": "document.body.style.backgroundColor = 'white';"
}
```

## Response Format

The response format depends on the `response_type` and `format` parameters:

- `"by_format"` (default):
   - Image formats (png, jpeg, webp): Binary image data with appropriate MIME type
   - PDF: Binary PDF data with `Content-Type: application/pdf`
   - HTML: HTML content with `Content-Type: text/html`

- `"empty"`: Returns 204 No Content

- `"json"`: Returns JSON object with base64-encoded content:
  ```json
  {
    "file": "base64_encoded_content",
    "format": "png"
  }
  ```

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
   - Consider using interaction steps for complex loading scenarios

2. **Resource Optimization**:
   - Use `block_media: true` when images aren't needed
   - Set appropriate viewport sizes to minimize memory usage
   - Use the `selector` parameter to capture specific elements when possible

3. **Error Handling**:
   - Implement exponential backoff for retry logic
   - Handle 429 responses properly when rate limiting is enabled
   - Use appropriate timeouts for your use case

4. **Security**:
   - Keep your AUTH_TOKEN secure
   - Validate and sanitize all URLs before sending
   - Use HTTPS for all API requests
   - Be cautious with custom JavaScript injection

Remember to handle errors gracefully and implement appropriate retry logic in your applications. For high-volume usage, contact your Pixashot administrator to discuss rate limit adjustments and resource allocation.