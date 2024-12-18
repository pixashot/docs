---
title: Network Security
excerpt: Comprehensive guide to configuring HTTPS, proxies, and network security features in Pixashot.
meta:
  nav_order: 112
---

# Network Security

Pixashot implements comprehensive network security measures to ensure secure communication and resource access. This guide covers HTTPS configuration, proxy setup, and network security best practices.

## HTTPS Configuration

### Production Settings

```nginx
# Nginx security headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Content-Security-Policy "default-src 'self'; img-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval';" always;
```

### Environment Configuration

```bash
# Force HTTPS
export HTTPS_ONLY=true
export IGNORE_HTTPS_ERRORS=false

# Certificate configuration (Cloud Run)
gcloud run services update pixashot \
  --region=us-central1 \
  --ingress=internal-and-cloud-load-balancing
```

## Proxy Configuration

### Basic Setup

```bash
# Proxy settings
export PROXY_SERVER="proxy.example.com"
export PROXY_PORT="8080"
export PROXY_USERNAME="proxy_user"
export PROXY_PASSWORD="secure_proxy_pass"
```

### Authentication Support

```python
proxy_config = {
    'server': f'{config.PROXY_SERVER}:{config.PROXY_PORT}',
    'username': config.PROXY_USERNAME,
    'password': config.PROXY_PASSWORD
}
```

## Network Controls

### Resource Restrictions

```bash
# Block unwanted resources
{
    "block_media": true,
    "ignore_https_errors": false,
    "use_popup_blocker": true,
    "use_cookie_blocker": true
}
```

### Domain Restrictions

```python
ALLOWED_DOMAINS = ['example.com', 'trusted-domain.com']
BLOCKED_DOMAINS = ['malicious.com']
BLOCKED_IP_RANGES = [
    '10.0.0.0/8',      # RFC 1918
    '172.16.0.0/12',   # RFC 1918
    '192.168.0.0/16',  # RFC 1918
]
```

## Security Headers

```python
@app.after_request
def security_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['Content-Security-Policy'] = "default-src 'self'"
    return response
```

## Next Steps

- Review [Security Best Practices](best-practices.md)
- Configure [Authentication](authentication.md)
- Set up [Monitoring](../deployment/cloud-run.md)