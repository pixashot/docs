---
title: API Reference
excerpt: Complete API reference documentation for Pixashot, including authentication, endpoints, error handling, and rate limiting.
meta:
    nav_order: 70
---

# API Reference

Pixashot provides a RESTful API for capturing high-quality screenshots of web pages. This reference covers all API endpoints, request options, and response formats.

## Base URL

```
https://your-pixashot-instance/
```

## Authentication

Pixashot uses token-based authentication. Include your token in all requests:

```bash
curl -X POST https://your-pixashot-instance/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'
```

### Authentication Methods

1. **Bearer Token**
    - Include in `Authorization` header
    - Required for all API requests
    - Format: `Bearer your_token`

2. **Signed URLs**
    - Alternative to header authentication
    - Includes expiration timestamp
    - Useful for client-side requests

## Available Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/capture` | POST | Capture a screenshot |
| `/health` | GET | Service health status |
| `/health/live` | GET | Basic liveness check |
| `/health/ready` | GET | Readiness check |

## Common HTTP Status Codes

- `200`: Success
- `400`: Invalid request parameters
- `401`: Missing authentication
- `403`: Invalid authentication
- `429`: Rate limit exceeded
- `500`: Server error

## Rate Limiting

Rate limits are configured server-side:

```bash
# Environment configuration
RATE_LIMIT_ENABLED=true
RATE_LIMIT_CAPTURE="5 per second"
```

Rate limit headers:
```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 4
X-RateLimit-Reset: 1635724800
```

## Response Formats

Pixashot supports multiple response formats:

- `png`: Default format, lossless with alpha channel
- `jpeg`: Configurable quality compression
- `webp`: Modern format with superior compression
- `pdf`: Document format with customizable paper size
- `html`: Complete page source capture

## Next Steps

- [Capture Endpoint](capture-endpoint.md): Main screenshot endpoint
- [Request Options](request-options.md): Available parameters
- [Response Handling](response-handling.md): Response formats
- [Rate Limiting](rate-limiting.md): Request throttling