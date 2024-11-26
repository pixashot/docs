---
title: Configuring Pixashot
excerpt: Complete guide to configuring Pixashot through environment variables and runtime options.
meta:
    nav_order: 23
---

# Configuring Pixashot

This guide covers all configuration options available in Pixashot, from basic settings to advanced customization.

## Environment Variables

### Required Settings

```bash
# Authentication
AUTH_TOKEN=your_secure_token    # Required for API authentication
PORT=8080                       # Server port (default: 8080)
```

### Worker Configuration

```bash
# Process Management
WORKERS=4                       # Number of worker processes (default: 4)
MAX_REQUESTS=1000              # Maximum requests per worker before restart (default: 1000)
KEEP_ALIVE=300                 # Keep-alive timeout in seconds (default: 300)
```

### Browser Extensions

```bash
# Feature Toggles
USE_POPUP_BLOCKER=true         # Enable/disable popup blocker (default: true)
USE_COOKIE_BLOCKER=true        # Enable/disable cookie consent blocker (default: true)
```

### Rate Limiting

```bash
# Rate Limiting Configuration
RATE_LIMIT_ENABLED=true        # Enable/disable rate limiting (default: false)
RATE_LIMIT_CAPTURE="1 per second"      # Rate limit for capture endpoint
RATE_LIMIT_SIGNED="5 per second"       # Rate limit for signed URLs
```

### Caching

```bash
# Response Caching
CACHE_MAX_SIZE=1000            # Maximum size for response caching (default: 0, disabled)
```

### Proxy Configuration

```bash
# Proxy Settings
PROXY_SERVER="proxy.example.com"    # Proxy server address
PROXY_PORT="8080"                   # Proxy server port
PROXY_USERNAME="user"               # Proxy authentication username
PROXY_PASSWORD="pass"               # Proxy authentication password
```

## Configuration Examples

### Basic Development Setup

```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN="your_secure_token" \
  -e WORKERS=2 \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest
```

### Production Setup

```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN="your_secure_token" \
  -e WORKERS=4 \
  -e MAX_REQUESTS=1000 \
  -e KEEP_ALIVE=300 \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  -e RATE_LIMIT_ENABLED=true \
  -e RATE_LIMIT_CAPTURE="5 per second" \
  -e CACHE_MAX_SIZE=1000 \
  --memory=4g \
  gpriday/pixashot:latest
```

### High-Traffic Setup

```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN="your_secure_token" \
  -e WORKERS=8 \
  -e MAX_REQUESTS=2000 \
  -e KEEP_ALIVE=300 \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  -e RATE_LIMIT_ENABLED=true \
  -e RATE_LIMIT_CAPTURE="10 per second" \
  -e CACHE_MAX_SIZE=5000 \
  --memory=8g \
  --cpus=4 \
  gpriday/pixashot:latest
```

## Resource Allocation Guidelines

### Memory Configuration

Memory requirements vary based on usage:
```
Light usage:   2GB minimum
Medium usage:  4GB recommended
Heavy usage:   8GB+ recommended
```

### CPU Allocation

CPU cores needed for different scenarios:
```
Development:   1 core
Production:    2+ cores
High traffic:  4+ cores
```

### Worker Configuration

Worker count guidelines:
```
Development:   WORKERS=2
Production:    WORKERS=4-8
High traffic:  WORKERS=8-16
```

## Template System

Pixashot supports templates for common capture scenarios. Templates are defined in `templates.json`:

```json
{
    "mobile": {
        "window_width": 375,
        "window_height": 667,
        "pixel_density": 2.0,
        "user_agent_device": "mobile",
        "user_agent_platform": "ios"
    },
    "desktop_full": {
        "window_width": 1920,
        "window_height": 1080,
        "full_page": true,
        "format": "png",
        "pixel_density": 2.0
    },
    "article_pdf": {
        "format": "pdf",
        "pdf_format": "A4",
        "pdf_print_background": true,
        "full_page": true,
        "wait_for_network": "idle",
        "block_media": true
    }
}
```

## Browser Context Settings

The browser context is shared across all requests and configured through environment variables:

```bash
# Browser Configuration
USE_POPUP_BLOCKER=true         # Controls popup blocking
USE_COOKIE_BLOCKER=true        # Controls cookie consent blocking
```

## Rate Limiting Configuration

Rate limiting can be configured using these patterns:

```bash
# Rate limit patterns
RATE_LIMIT_CAPTURE="1 per second"    # One request per second
RATE_LIMIT_CAPTURE="60 per minute"   # 60 requests per minute
RATE_LIMIT_CAPTURE="1000 per hour"   # 1000 requests per hour
```

## Security Configuration

### Authentication Settings

```bash
# Generate secure token
export AUTH_TOKEN=$(openssl rand -hex 32)

# For signed URLs
export URL_SIGNING_SECRET=$(openssl rand -hex 32)
```

### Network Security

```bash
# HTTPS Configuration
export HTTPS_ONLY=true              # Force HTTPS
export IGNORE_HTTPS_ERRORS=false    # Strict HTTPS validation
```

## Monitoring Configuration

### Health Checks

Configure health check endpoints:
```bash
# Health check paths
/health          # Basic health status
/health/live     # Liveness probe
/health/ready    # Readiness probe
```

### Logging

```bash
# Logging Configuration
LOG_LEVEL=info                 # Logging level (debug, info, warning, error)
LOG_FORMAT=json               # Log format (json or text)
```

## Performance Tuning

### Caching Configuration

```bash
# Cache Settings
CACHE_MAX_SIZE=1000           # Number of responses to cache
CACHE_TTL=3600               # Cache TTL in seconds
```

### Request Timeouts

```bash
# Timeout Configuration
KEEP_ALIVE=300               # Connection keep-alive timeout
MAX_REQUEST_TIME=30          # Maximum request processing time
```

## Environment-Specific Configurations

### Development

```bash
# Development Settings
export AUTH_TOKEN="dev_token"
export WORKERS=2
export RATE_LIMIT_ENABLED=false
export LOG_LEVEL=debug
```

### Staging

```bash
# Staging Settings
export AUTH_TOKEN="secure_token"
export WORKERS=4
export RATE_LIMIT_ENABLED=true
export RATE_LIMIT_CAPTURE="5 per second"
export LOG_LEVEL=info
```

### Production

```bash
# Production Settings
export AUTH_TOKEN="very_secure_token"
export WORKERS=8
export RATE_LIMIT_ENABLED=true
export RATE_LIMIT_CAPTURE="10 per second"
export CACHE_MAX_SIZE=5000
export LOG_LEVEL=warning
```

## Configuration Validation

Verify your configuration with:

```bash
# Test configuration
curl http://localhost:8080/health

# Check rate limits
curl -I -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token"

# Verify caching
curl http://localhost:8080/health | grep cache_size
```

## Next Steps

After configuration:
1. Review [Best Practices](best-practices.md) for optimization tips
2. Check [Security](../security/index.md) for security hardening
3. Configure [Monitoring](../deployment/monitoring.md)
4. Set up [Error Handling](../api-reference/error-handling.md)

For deployment-specific configurations, see our [Deployment Guide](../deployment/index.md).