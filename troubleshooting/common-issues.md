---
title: Common Issues
excerpt: Detailed solutions for frequently encountered problems when using Pixashot, including screenshots, timeouts, and resource issues.
meta:
  nav_order: 171
---

# Common Issues and Solutions

## Screenshot Issues

### Blank or White Screenshots
**Problem**: Screenshots appear blank or white.
**Solutions**:
1. Increase wait timeout:
```json
{
    "url": "https://example.com",
    "wait_for_timeout": 10000,
    "wait_for_network": "idle"
}
```

2. Check JavaScript errors:
```bash
python diagnostic.py https://example.com
```

3. Verify content loading:
```json
{
    "wait_for_selector": "#main-content",
    "wait_for_network": "idle"
}
```

### Missing Content
**Problem**: Screenshots missing dynamic content.
**Solutions**:
1. Use appropriate wait strategy:
```json
{
    "wait_for_network": "idle",
    "wait_for_animation": true,
    "delay_capture": 1000
}
```

2. Add custom wait logic:
```json
{
    "custom_js": "
        await new Promise(resolve => {
            const observer = new MutationObserver(() => {
                if (document.querySelector('#dynamic-content')) {
                    observer.disconnect();
                    resolve();
                }
            });
            observer.observe(document.body, { childList: true, subtree: true });
        });
    "
}
```

## Performance Issues

### Memory Errors
**Problem**: Out of memory errors.
**Solutions**:
1. Adjust worker configuration:
```bash
export WORKERS=2
export MAX_REQUESTS=500
```

2. Enable media blocking:
```json
{
    "block_media": true,
    "format": "jpeg",
    "image_quality": 80
}
```

3. Monitor memory usage:
```bash
curl http://your-instance/health | jq .checks.memory_usage_mb
```

### Slow Captures
**Problem**: Screenshot captures taking too long.
**Solutions**:
1. Use "mostly_idle" network strategy:
```json
{
    "wait_for_network": "mostly_idle",
    "wait_for_timeout": 5000
}
```

2. Block unnecessary resources:
```json
{
    "block_media": true,
    "custom_js": "document.querySelectorAll('video, iframe').forEach(el => el.remove());"
}
```

## Network Issues

### SSL/TLS Errors
**Problem**: SSL certificate validation fails.
**Solutions**:
1. Enable HTTPS error ignoring:
```json
{
    "ignore_https_errors": true
}
```

2. Configure proper certificates:
```bash
export SSL_CERT_PATH=/path/to/cert.pem
export SSL_KEY_PATH=/path/to/key.pem
```

### Proxy Issues
**Problem**: Cannot connect through proxy.
**Solutions**:
1. Verify proxy configuration:
```bash
export PROXY_SERVER="proxy.example.com"
export PROXY_PORT="8080"
export PROXY_USERNAME="user"
export PROXY_PASSWORD="pass"
```

2. Test proxy connection:
```bash
curl -v --proxy $PROXY_SERVER:$PROXY_PORT https://example.com
```

## Resource Management

### Context Crashes
**Problem**: Browser context crashes.
**Solutions**:
1. Monitor health status:
```bash
watch -n 5 'curl -s http://your-instance/health'
```

2. Reduce concurrent load:
```bash
export MAX_REQUESTS=500
export RATE_LIMIT_CAPTURE="1 per second"
```

### Worker Issues
**Problem**: Worker processes become unresponsive.
**Solutions**:
1. Adjust worker lifecycle:
```bash
export WORKERS=4
export MAX_REQUESTS=1000
export KEEP_ALIVE=300
```

2. Enable worker recycling:
```bash
export AUTO_RESTART=true
export RESTART_INTERVAL=3600
```

## Quick Solutions Table

| Issue | Check | Solution |
|-------|--------|----------|
| Blank screenshots | Network activity | Increase `wait_for_timeout` |
| Missing content | JavaScript console | Add content-specific wait |
| Memory errors | Memory usage | Reduce workers, enable blocking |
| Slow captures | Resource loading | Use `mostly_idle`, block media |
| SSL errors | Certificate validity | Enable error ignoring |
| Proxy issues | Proxy connectivity | Verify proxy configuration |

## Next Steps

If these solutions don't resolve your issue:
1. Use the [Debugging Guide](debugging.md)
2. Check the [FAQ](faq.md)
3. Review [Best Practices](../getting-started/best-practices.md)
4. Contact support with diagnostic output.