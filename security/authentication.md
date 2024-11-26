---
title: Authentication
excerpt: Detailed guide to Pixashot's authentication mechanisms, including token-based auth and signed URLs.
meta:
    nav_order: 111
---

# Authentication

Pixashot supports two authentication methods: Bearer Token Authentication and Signed URLs. This guide explains how to implement and use both approaches securely.

## Bearer Token Authentication

The primary authentication method using an HTTP Authorization header.

### Configuration

```bash
# Generate secure token
export AUTH_TOKEN=$(openssl rand -hex 32)

# Store in secret manager (Google Cloud example)
echo -n "your_secret_token" | \
  gcloud secrets create pixashot-auth-token \
  --data-file=- \
  --replication-policy="automatic"
```

### Usage

Include the token in the Authorization header:

```bash
curl -X POST https://your-instance/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png"
  }'
```

## Signed URLs

For scenarios where header modification isn't possible, use signed URLs with expiration.

### Python Implementation

```python
import time
import hmac
import hashlib
import base64
from urllib.parse import urlencode

def generate_signed_url(base_url, params, secret_key, expires_in=3600):
    """Generate a signed URL with expiration."""
    expiration = int(time.time()) + expires_in
    params['expires'] = expiration
    
    message = '&'.join(f"{k}={v}" for k, v in sorted(params.items()))
    signature = base64.urlsafe_b64encode(
        hmac.new(
            secret_key.encode('utf-8'),
            message.encode('utf-8'),
            hashlib.sha256
        ).digest()
    ).rstrip(b'=').decode('ascii')
    
    params['signature'] = signature
    return f"{base_url}?{urlencode(params)}"

# Usage
url = generate_signed_url(
    "https://your-instance/capture",
    {"url": "https://example.com", "format": "png"},
    "your_secret_key",
    expires_in=3600
)
```

## Rate Limiting

Configure rate limits to prevent abuse:

```bash
# Environment configuration
export RATE_LIMIT_ENABLED=true
export RATE_LIMIT_CAPTURE="5 per second"    # Capture endpoint
export RATE_LIMIT_SIGNED="10 per second"    # Signed URLs
```

## Security Recommendations

1. **Token Management**
    - Use cryptographically secure random tokens
    - Rotate tokens regularly
    - Store tokens securely
    - Use different tokens per environment

2. **URL Signing**
    - Keep signing keys secure
    - Use short expiration times
    - Validate all parameters
    - Monitor usage patterns

3. **Rate Limiting**
    - Enable rate limiting in production
    - Set appropriate limits
    - Monitor failed attempts
    - Implement gradual backoff

## Error Handling

Common authentication errors:

```json
{
    "status": "error",
    "message": "Authentication failed",
    "error_type": "AuthenticationError",
    "timestamp": "2024-01-01T12:00:00Z"
}
```

Status codes:
- 401: Missing authentication
- 403: Invalid token
- 429: Rate limit exceeded

## Next Steps

- Configure [Network Security](network-security.md)
- Review [Best Practices](best-practices.md)
- Set up [Monitoring](../deployment/monitoring.md)