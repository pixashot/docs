---
title: Response Handling
excerpt: Guide to handling Pixashot API responses, including success responses, error handling, and different response formats.
meta:
    nav_order: 93
---

# Response Handling

Guide to handling responses from the Pixashot API, including success states, errors, and different response formats.

## Response Types

Configure response type using the `response_type` parameter:

| Type | Description |
|------|-------------|
| `by_format` | Binary response based on format (default) |
| `json` | Base64 encoded response |
| `empty` | No content, status code only |

## Format-Specific Responses

### Image Formats (PNG/JPEG/WebP)
```http
HTTP/1.1 200 OK
Content-Type: image/png
Content-Disposition: attachment; filename=screenshot.png
```
Response body contains binary image data.

### PDF Format
```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename=screenshot.pdf
```
Response body contains binary PDF data.

### HTML Format
```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Disposition: attachment; filename=screenshot.html
```
Response body contains HTML content.

## JSON Response Format

When `response_type: "json"`:
```json
{
  "file": "base64_encoded_content",
  "format": "png"
}
```

## Error Responses

### Validation Error
```json
{
  "status": "error",
  "message": "Invalid parameters",
  "error_type": "ValidationError",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### Authentication Error
```json
{
  "status": "error",
  "message": "Invalid authentication token",
  "error_type": "AuthenticationError",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### Rate Limit Error
```json
{
  "status": "error",
  "message": "Rate limit exceeded",
  "error_type": "RateLimitError",
  "timestamp": "2024-01-01T12:00:00Z",
  "retry_after": 5
}
```

## Response Headers

### Standard Headers
```http
Content-Type: [mime-type]
Content-Disposition: attachment; filename=screenshot.[format]
Content-Length: [size]
```

### Rate Limit Headers
```http
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 4
X-RateLimit-Reset: 1635724800
```

## Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 204 | Success (empty response) |
| 400 | Invalid parameters |
| 401 | Missing authentication |
| 403 | Invalid authentication |
| 429 | Rate limit exceeded |
| 500 | Server error |

## Client Implementation

### JavaScript Example
```javascript
async function captureScreenshot(options) {
  try {
    const response = await fetch('/capture', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(options)
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message);
    }

    return response.blob();
  } catch (error) {
    console.error('Screenshot failed:', error);
    throw error;
  }
}
```

### Error Handling Best Practices

1. **Validate Status Code**
    - Check response.ok
    - Handle specific status codes
    - Parse error response body

2. **Rate Limiting**
    - Check rate limit headers
    - Implement backoff strategy
    - Queue requests if needed

3. **Timeout Handling**
    - Set appropriate timeouts
    - Implement retry logic
    - Handle network errors

4. **Response Validation**
    - Verify content type
    - Check response size
    - Validate format-specific content