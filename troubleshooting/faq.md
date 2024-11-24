---
title: Frequently Asked Questions
excerpt: Common questions and answers about Pixashot usage, configuration, deployment, and troubleshooting.
meta:
    nav_order: 93
---

# Frequently Asked Questions

## Screenshots

### Why is my screenshot blank?
- Page hasn't finished loading (increase `wait_for_timeout`)
- JavaScript errors (check diagnostic tool output)
- Network connectivity issues
- Resource loading failures

### How do I capture dynamic content?
```json
{
    "wait_for_network": "idle",
    "wait_for_animation": true,
    "custom_js": "await new Promise(r => setTimeout(r, 1000))"
}
```

### What's the maximum screenshot size?
- Maximum width: 16384px
- Maximum height: 16384px
- Maximum file size: Limited by memory

## Configuration

### How do I configure workers?
```bash
# Production settings
export WORKERS=4                  # CPU cores
export MAX_REQUESTS=1000         # Requests per worker
export KEEP_ALIVE=300           # Seconds
```

### How do I enable caching?
```bash
# Enable response caching
export CACHE_MAX_SIZE=1000      # Number of responses
export CACHE_TTL=3600          # Seconds
```

### How do I handle rate limiting?
```bash
# Configure rate limits
export RATE_LIMIT_ENABLED=true
export RATE_LIMIT_CAPTURE="5 per second"
```

## Performance

### What are the memory requirements?
- Minimum: 2GB RAM
- Recommended: 4GB RAM
- Production: 8GB+ RAM

### How can I optimize performance?
1. Enable media blocking
2. Use appropriate wait strategies
3. Configure worker count
4. Enable caching
5. Set proper timeouts

### How do I monitor resource usage?
```bash
# Health check endpoint
curl http://your-instance/health

# Memory monitoring
docker stats pixashot
```

## Deployment

### Which platforms are supported?
- Docker (recommended for development)
- Google Cloud Run (recommended for production)
- AWS ECS/Fargate
- Azure Container Services
- Any Docker-compatible host

### How do I update Pixashot?
```bash
# Docker
docker pull gpriday/pixashot:latest

# Cloud Run
gcloud run deploy pixashot --image gpriday/pixashot:latest
```

## Security

### How do I secure my instance?
1. Set secure authentication token
2. Enable HTTPS
3. Configure rate limiting
4. Use private networking
5. Monitor access logs

### Is caching secure?
- Cache is per-instance
- No cross-request data sharing
- Memory-only caching
- Regular cache clearing

## Technical Limits

### Browser Context
- Single shared context
- Multiple worker processes
- Automatic cleanup
- Resource pooling

### Network
- Proxy support
- Custom headers
- SSL/TLS handling
- Request timeouts

## Support

### Where can I get help?
1. Documentation
2. GitHub Issues
3. Email Support
4. Community Forums

### How do I report bugs?
1. Run diagnostic tool
2. Collect logs
3. Create GitHub issue
4. Include reproduction steps

## Next Steps

- Review [Common Issues](common-issues.md)
- Check [Debugging Guide](debugging.md)
- Follow [Best Practices](../getting-started/best-practices.md)
- Contact support if needed