---
excerpt: "Comprehensive security guide for Pixashot, covering authentication, data protection, deployment security, and best practices for secure screenshot capture."
published_at: true
---

# Security

Security is a core feature of Pixashot. This guide covers essential security aspects of deploying, configuring, and using Pixashot in production environments.

## Authentication

### Bearer Token Authentication

Pixashot uses token-based authentication for API access:

```bash
# Configuration
export AUTH_TOKEN="your_secure_token_here"
```

Best practices for token management:
- Use cryptographically secure random tokens (minimum 32 bytes)
- Store tokens in secure environment variables or secret management systems
- Rotate tokens regularly (recommended: every 90 days)
- Never commit tokens to version control
- Use different tokens for development and production

Example token generation:
```bash
# Generate a secure random token
openssl rand -hex 32
```

### Signed URLs

For scenarios requiring temporary access:

```python
import hmac
import hashlib
import time
import base64

def generate_signed_url(base_url, params, secret_key, expires_in=3600):
    """Generate a signed URL with expiration."""
    expiration = int(time.time()) + expires_in
    params['expires'] = expiration
    
    # Create signature
    message = '&'.join(f"{k}={v}" for k, v in sorted(params.items()))
    signature = base64.urlsafe_b64encode(
        hmac.new(
            secret_key.encode('utf-8'),
            message.encode('utf-8'),
            hashlib.sha256
        ).digest()
    ).rstrip(b'=').decode('ascii')
    
    params['signature'] = signature
    query_string = '&'.join(f"{k}={v}" for k, v in params.items())
    return f"{base_url}?{query_string}"
```

## Network Security

### HTTPS Configuration

Required security headers:
```nginx
# Nginx configuration
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Content-Security-Policy "default-src 'self'; img-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';" always;
```

### Rate Limiting

Configure rate limits to prevent abuse:

```bash
# Environment configuration
export RATE_LIMIT_ENABLED="true"
export RATE_LIMIT_CAPTURE="1 per second"    # Capture endpoint limit
export RATE_LIMIT_SIGNED="5 per second"     # Signed URL limit
```

Rate limit response headers:
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
X-RateLimit-Reset: 1635724800
```

### Proxy Configuration

Secure proxy settings:
```bash
# Environment configuration
export PROXY_SERVER="proxy.example.com"
export PROXY_PORT="8080"
export PROXY_USERNAME="proxy_user"
export PROXY_PASSWORD="secure_proxy_password"
```

## Resource Security

### Browser Context Security

Pixashot implements several browser-level security measures:

1. **Sandboxed Execution**:
   ```javascript
   // Browser launch arguments
   args=[
       '--disable-features=site-per-process',
       '--no-sandbox',
       '--disable-setuid-sandbox',
       '--disable-dev-shm-usage',
   ]
   ```

2. **Resource Control**:
   ```json
   {
     "block_media": true,
     "ignore_https_errors": false,
     "use_popup_blocker": true,
     "use_cookie_blocker": true
   }
   ```

3. **Custom Header Security**:
   ```python
   # Input validation for custom headers
   @model_validator(mode='after')
   def validate_custom_headers(self) -> 'CaptureRequest':
       if self.custom_headers:
           for header_name in self.custom_headers.keys():
               if not isinstance(header_name, str) or ':' in header_name:
                   raise ValueError(f"Invalid header name: {header_name}")
       return self
   ```

### URL Security

Built-in URL validation and security checks:

```python
def validate_url(url: str) -> Tuple[bool, Optional[str]]:
    """
    Validate URL against security rules.
    Returns (is_valid, error_message if any)
    """
    # Validate URL format
    try:
        parsed = urlparse(url)
        if not all([parsed.scheme, parsed.netloc]):
            return False, "Invalid URL format"
        
        # Check for allowed schemes
        if parsed.scheme not in {'http', 'https'}:
            return False, "Invalid URL scheme"
            
        # Block internal/private networks
        if not is_public_ip(parsed.netloc):
            return False, "Internal network access not allowed"
            
        return True, None
        
    except Exception as e:
        return False, str(e)
```

## Data Privacy

### Request Data Protection

1. **Memory Management**:
   ```python
   # Secure cleanup after capture
   try:
       screenshot_data = await capture_screenshot(options)
       return screenshot_data
   finally:
       await cleanup_resources()
   ```

2. **Temporary File Handling**:
   ```python
   def secure_tempfile():
       """Create secure temporary file with proper permissions."""
       fd, path = tempfile.mkstemp(prefix='pixashot_', suffix='.tmp')
       os.chmod(path, 0o600)  # Restrictive permissions
       return fd, path
   ```

### Response Security

1. **Content Security**:
   ```python
   @app.after_request
   def secure_response(response):
       """Add security headers to all responses."""
       response.headers['X-Content-Type-Options'] = 'nosniff'
       response.headers['X-Frame-Options'] = 'DENY'
       return response
   ```

2. **Error Handling**:
   ```python
   def handle_error(error):
       """Secure error handling without leaking details."""
       return {
           'status': 'error',
           'message': 'An error occurred',
           'error_id': generate_error_id()  # For logging correlation
       }, error.code
   ```

## Deployment Security

### Docker Security

Secure Dockerfile configuration:
```dockerfile
# Run as non-root user
RUN useradd -m appuser && \
    mkdir -p /tmp/screenshots /app/data /app/logs && \
    chown -R appuser:appuser /app /tmp/screenshots /app/data /app/logs

USER appuser

# Security-related arguments
ENV NODE_OPTIONS="--no-experimental-fetch --no-warning"
```

### Cloud Run Security

Secure Cloud Run configuration:
```yaml
steps:
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - '${_SERVICE_NAME}'
      - '--image'
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPOSITORY}/${_SERVICE_NAME}:${_TAG}'
      - '--platform'
      - 'managed'
      - '--ingress'
      - 'internal-and-cloud-load-balancing'
      - '--memory'
      - '${_MEMORY}'
      - '--cpu'
      - '${_CPU}'
      - '--set-secrets'
      - '/AUTH_TOKEN=pixashot-auth-token:latest'
```

## Security Checklist

- [ ] Configure authentication token
- [ ] Enable HTTPS with proper certificates
- [ ] Set up rate limiting
- [ ] Configure proxy settings (if needed)
- [ ] Enable security headers
- [ ] Set up monitoring and logging
- [ ] Implement proper error handling
- [ ] Configure resource limits
- [ ] Set up automated security updates
- [ ] Implement access controls
- [ ] Configure backup and recovery
- [ ] Set up incident response plan

## Monitoring and Auditing

1. **Health Monitoring**:
   ```bash
   # Check system health
   curl https://your-instance/health
   
   # Check detailed status
   curl https://your-instance/health/ready
   ```

2. **Security Logging**:
   ```python
   logger.info('Capture request', extra={
       'url': request.url,
       'client_ip': request.remote_addr,
       'user_agent': request.headers.get('User-Agent'),
       'request_id': request.headers.get('X-Request-ID')
   })
   ```

Remember:
- Regularly update all components and dependencies
- Monitor security advisories
- Conduct regular security audits
- Maintain incident response procedures
- Keep security documentation up to date

For more information about specific security features, refer to the [API Reference](api-reference.md) and [Deployment Guide](deployment.md).