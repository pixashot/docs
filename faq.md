# Frequently Asked Questions

## Table of Contents

- [General Questions](#general-questions)
- [Installation & Setup](#installation--setup)
- [Usage & Features](#usage--features)
- [Performance & Optimization](#performance--optimization)
- [Security & Authentication](#security--authentication)
- [Deployment & Hosting](#deployment--hosting)
- [Billing & Pricing](#billing--pricing)
- [Errors & Troubleshooting](#errors--troubleshooting)

## General Questions

### What is Pixashot?
Pixashot is a high-performance web screenshot service that captures pixel-perfect screenshots of web pages. It supports multiple formats, custom viewports, and advanced features like JavaScript injection and geolocation spoofing.

### What formats does Pixashot support?
Pixashot supports multiple output formats:
- PNG (lossless with transparency)
- JPEG (configurable quality)
- WebP (modern format with superior compression)
- PDF (with customizable paper sizes and options)
- HTML (full page source capture)

### Can I use Pixashot in production?
Yes! Pixashot is designed for production use with features like:
- Rate limiting
- Authentication
- Health monitoring
- Resource optimization
- High availability options

## Installation & Setup

### How do I install Pixashot?

**Docker**:
```bash
docker pull gpriday/pixashot:latest
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  gpriday/pixashot:latest
```

**Google Cloud Run (Recommended)**:
```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1
```

### What are the minimum system requirements?
- CPU: 1 core
- RAM: 2GB minimum 
- Storage: 1GB for installation
- Operating System: Any Docker-compatible OS

### How do I update Pixashot?
```bash
# Docker
docker pull gpriday/pixashot:latest

# Cloud Run
gcloud run deploy pixashot --image gpriday/pixashot:latest
```

## Usage & Features

### How do I capture a simple screenshot?
```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png"
  }'
```

### How do I capture a full-page screenshot?
```json
{
  "url": "https://example.com",
  "full_page": true,
  "format": "png",
  "wait_for_network": "idle"
}
```

### Can I capture a specific element?
Yes, use the `selector` parameter:
```json
{
  "url": "https://example.com",
  "selector": "#main-content",
  "format": "png"
}
```

### Does Pixashot support mobile viewport simulation?
Yes, configure the viewport and pixel density:
```json
{
  "url": "https://example.com",
  "window_width": 375,
  "window_height": 812,
  "pixel_density": 2.0,
  "format": "png"
}
```

### How do I handle cookie consent popups?
Enable the cookie consent blocker:
```bash
docker run -e USE_COOKIE_BLOCKER=true ...
```

## Performance & Optimization

### How can I improve capture speed?
1. Use `block_media: true` when images aren't needed
2. Set appropriate viewport sizes
3. Use `wait_for_network: "mostly_idle"` instead of `"idle"`
4. Enable caching for repeated captures
5. Use the minimum necessary pixel density

### What's the maximum screenshot size?
The default maximum height is 16384 pixels. For width, we recommend staying under 5000 pixels for optimal performance.

### Does Pixashot support rate limiting?
Yes, configure rate limits with:
```bash
export RATE_LIMIT_ENABLED=true
export RATE_LIMIT_CAPTURE="10 per minute"
```

## Security & Authentication

### How do I secure my Pixashot instance?
1. Always use an authentication token:
   ```bash
   export AUTH_TOKEN=$(openssl rand -hex 32)
   ```

2. Enable HTTPS
3. Configure rate limiting
4. Use the latest version
5. Follow [security best practices](security.md)

### Can I use temporary access tokens?
Yes, use signed URLs:
```python
from request_auth import generate_signed_url

url = generate_signed_url(
    "https://your-instance/capture",
    {"url": "https://example.com"},
    "your_secret_key",
    expires_in=3600
)
```

## Deployment & Hosting

### Where can I deploy Pixashot?
- Google Cloud Run (recommended)
- AWS (ECS, EKS, or App Runner)
- Azure Container Services
- DigitalOcean App Platform
- Any Docker-compatible host

### How much does it cost to run?
Costs vary by platform and usage, but for reference:
- Google Cloud Run free tier includes:
    - 2 million requests/month
    - 360,000 vCPU-seconds
    - 180,000 GiB-seconds
- Most users stay within free tier limits for moderate usage

### Can I run Pixashot behind a proxy?
Yes, configure proxy settings:
```bash
export PROXY_SERVER="proxy.example.com"
export PROXY_PORT="8080"
export PROXY_USERNAME="user"
export PROXY_PASSWORD="pass"
```

## Errors & Troubleshooting

### Why is my screenshot blank/white?
Common causes:
1. Page not fully loaded (increase `wait_for_timeout`)
2. JavaScript errors (check console logs)
3. Content requires interaction
4. Network issues

See the [Troubleshooting Guide](troubleshooting.md) for solutions.

### How do I debug capturing issues?
1. Enable debug logging:
   ```bash
   export LOG_LEVEL=debug
   ```

2. Use the diagnostic script:
   ```bash
   python diagnostic.py https://example.com
   ```

3. Check health endpoints:
   ```bash
   curl http://your-instance/health
   curl http://your-instance/health/ready
   ```

### What does "Context creation failed" mean?
This usually indicates resource issues:
1. Check memory usage
2. Verify browser health
3. Ensure proper permissions
4. Check container resources

## Additional Resources

- [API Documentation](api-reference.md)
- [Deployment Guide](deployment.md)
- [Example Use Cases](example-use-cases.md)
- [Security Guide](security.md)
- [GitHub Repository](https://github.com/pixashot/pixashot)

## Still Have Questions?

1. Check our [documentation](https://pixashot.com/docs)
2. Search [GitHub Issues](https://github.com/pixashot/pixashot/issues)
3. Contact support: support@pixashot.com