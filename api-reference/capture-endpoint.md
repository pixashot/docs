---
title: Capture Endpoint
excerpt: Detailed documentation for Pixashot's main screenshot capture endpoint, including request examples and error handling.
meta:
  nav_order: 91
---

# Capture Endpoint

The `/capture` endpoint is the primary interface for taking screenshots with Pixashot.

## Endpoint Details

```
POST /capture
```

## Basic Usage

```bash
curl -X POST https://your-pixashot-instance/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true,
    "window_width": 1280,
    "window_height": 720
  }'
```

## Common Request Examples

### Full Page Screenshot
```json
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "wait_for_network": "idle"
}
```

### Element Capture
```json
{
  "url": "https://example.com",
  "selector": "#main-content",
  "format": "png",
  "wait_for_selector": "#main-content"
}
```

### PDF Generation
```json
{
  "url": "https://example.com",
  "format": "pdf",
  "pdf_format": "A4",
  "pdf_print_background": true,
  "full_page": true
}
```

### Mobile Viewport
```json
{
  "url": "https://example.com",
  "window_width": 375,
  "window_height": 812,
  "pixel_density": 2.0,
  "format": "png"
}
```

## Error Responses

### Invalid Parameters
```json
{
  "status": "error",
  "message": "Invalid request parameters",
  "error_type": "ValidationError",
  "error_details": "Either url or html_content must be provided"
}
```

### Authentication Error
```json
{
  "status": "error",
  "message": "Invalid authentication token",
  "error_type": "AuthenticationError"
}
```

### Rate Limit Error
```json
{
  "status": "error",
  "message": "Rate limit exceeded",
  "error_type": "RateLimitError",
  "retry_after": 5
}
```

## Timeout Handling

Default timeouts:
- Network wait: 30 seconds
- Page load: 30 seconds
- Element wait: 30 seconds

Configure custom timeouts:
```json
{
  "url": "https://example.com",
  "format": "png",
  "wait_for_timeout": 60000,
  "wait_for_network": "idle"
}
```

## Response Headers

Successful responses include:
```
Content-Type: image/png
Content-Disposition: attachment; filename=screenshot.png
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 4
X-RateLimit-Reset: 1635724800
```

## Best Practices

1. **Error Handling**
    - Implement retry logic with backoff
    - Handle rate limit responses
    - Check response status codes

2. **Performance**
    - Use `wait_for_network: "mostly_idle"` for faster captures
    - Enable `block_media` when images aren't needed
    - Set appropriate viewport sizes

3. **Resource Management**
    - Monitor rate limits
    - Handle timeouts gracefully
    - Implement proper error handling

## Security Notes

- Always use HTTPS in production
- Keep authentication tokens secure
- Validate and sanitize URLs
- Use proper error handling