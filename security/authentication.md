---
title: Authentication
excerpt: Detailed guide to Pixashot's authentication mechanisms, including token-based auth and signed URLs.
meta:
  nav_order: 111
---

# Authentication

Pixashot supports two authentication methods: Bearer Token Authentication and Signed URLs. This guide explains how to
implement and use both approaches securely.

## Bearer Token Authentication

The primary authentication method using an HTTP Authorization header.

### Configuration

```bash
# Generate secure token
export AUTH_TOKEN=$(openssl rand -hex 32)

# Store in secret manager (Google Cloud example)
echo -n "$AUTH_TOKEN" | \
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

### Key Concepts

Signed URLs provide a way to grant temporary access to the `/capture` endpoint without needing to set the
`Authorization` header. They are particularly useful for:

* **Client-side requests:** Where setting custom headers might be difficult or impossible.
* **Sharing temporary access:** When you want to provide limited-time access to generate a screenshot without sharing
  your main API token.
* **Direct-to-cloud storage uploads:** Securely uploading screenshots directly from the client side to your cloud
  storage.

### Implementation

A signed URL includes the following parameters:

* `expires`: A Unix timestamp indicating when the URL should expire.
* `signature`: An HMAC-SHA256 signature generated using a secret key.

#### Server-Side Signature Verification

Here's how the server verifies a signed URL in python:

```python
from urllib.parse import parse_qs, urlparse
import time
import hmac
import hashlib
import base64
from src.exceptions import AuthenticationError, SignatureExpiredError, InvalidSignatureError

def verify_signed_url(query_params, secret_key):
    """
    Verifies the signature and expiration of a signed URL.

    Args:
        query_params: A dictionary of query parameters.
        secret_key: The secret key used for signing.

    Raises:
        AuthenticationError: If the signature is invalid or expired.
    """
    signature = query_params.get('signature', [None])[0]
    expires = query_params.get('expires', [None])[0]

    if not signature:
        raise InvalidSignatureError("Missing signature")

    if expires:
        expires = int(expires)
        if time.time() > expires:
            raise SignatureExpiredError("Signature has expired")

    params_to_sign = {k: v[0] for k, v in query_params.items() if k not in ['signature']}
    expected_signature = generate_signature(params_to_sign, secret_key)

    if not hmac.compare_digest(signature, expected_signature):
        raise InvalidSignatureError("Invalid signature")

def generate_signature(params, secret_key):
    """Generates an HMAC-SHA256 signature for the given parameters."""
    param_string = '&'.join(f"{k}={v}" for k, v in sorted(params.items()))
    message = param_string.encode('utf-8')
    return base64.urlsafe_b64encode(
        hmac.new(secret_key.encode('utf-8'), message, hashlib.sha256).digest()
    ).rstrip(b'=').decode('ascii')

def is_authenticated(request):
    """Checks if a request is authenticated using either a bearer token or a signed URL."""
    from src.config import config  # Import config locally to avoid circular dependencies
    if not config.AUTH_TOKEN:
        return True

    auth_header = request.headers.get('Authorization')
    if verify_auth_token(auth_header):
        return True

    parsed_url = urlparse(request.url)
    query_params = parse_qs(parsed_url.query)

    try:
        verify_signed_url(query_params, config.URL_SIGNING_SECRET)
        return True
    except AuthenticationError:
        return False
```

**Important**: Ensure you store your `URL_SIGNING_SECRET` securely, preferably in a secrets management service, similar
to how you store your `AUTH_TOKEN`.

#### Python Example: Generating a Signed URL

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

# Example Usage:
base_url = "https://your-instance/capture"  # Replace with your instance URL
params = {"url": "https://example.com", "format": "png"}
secret_key = "your_secret_key"  # Replace with your actual secret key

signed_url = generate_signed_url(base_url, params, secret_key, expires_in=3600)
print(signed_url)
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
    * Use cryptographically secure random tokens
    * Rotate tokens regularly
    * Store tokens securely
    * Use different tokens per environment

2. **URL Signing**
    * Keep signing keys secure and separate from auth tokens
    * Use short expiration times (e.g., 5-15 minutes)
    * Validate all parameters on the server-side, even if they are signed.
    * Monitor usage patterns for suspicious activity

3. **Rate Limiting**
    * Enable rate limiting in production
    * Set appropriate limits based on expected usage and infrastructure capacity
    * Monitor failed attempts and consider blocking IPs with excessive failures.
    * Implement gradual backoff on the client-side when encountering rate limits.

## Error Handling

The API will return specific error codes and messages for authentication failures:

| Error                  | Status Code | Description                                       |
|------------------------|:------------|:--------------------------------------------------|
| Missing authentication | 401         | No `Authorization` header or `signature` present. |
| Invalid token          | 403         | Incorrect or expired bearer token.                |
| Invalid signature      | 403         | Incorrect signature in signed URL.                |
| Expired signature      | 403         | Signed URL has expired.                           |
| Rate limit exceeded    | 429         | Too many requests from the client.                |

### Common Error Response Structure

```json
{
  "status": "error",
  "message": "Authentication failed",
  "error_type": "AuthenticationError",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

* `error_type` will be one of: `AuthenticationError`, `InvalidSignatureError`, `SignatureExpiredError`, or
  `RateLimitError`.

## Next Steps

- Configure [Network Security](network-security.md)
- Review [Best Practices](best-practices.md)
- Set up [Monitoring](../deployment/cloud-run.md)