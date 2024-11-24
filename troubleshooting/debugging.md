---
title: Debugging Guide
excerpt: Tools and techniques for debugging Pixashot issues, including diagnostic tools, logging configuration, and troubleshooting procedures.
meta:
    nav_order: 92
---

# Debugging Guide

## Diagnostic Tool

Pixashot includes a built-in diagnostic tool for troubleshooting:

```bash
# Basic usage
python diagnostic.py https://example.com

# Output includes:
# - Network requests
# - JavaScript errors
# - Resource timing
# - Memory usage
# - Screenshot result
```

## Logging Configuration

Enable debug logging for detailed output:

```bash
# Docker deployment
docker run -e LOG_LEVEL=debug pixashot

# Environment variable
export LOG_LEVEL=debug
```

## Browser Context Debugging

Monitor browser context status:

```python
# Get context metrics
curl http://your-instance/health | jq '.checks.browser_context'

# Memory usage per page
curl http://your-instance/health | jq '.checks.memory_per_page_mb'
```

## Network Analysis

Debug network-related issues:

```bash
# Watch network activity
python diagnostic.py --network-trace https://example.com

# Check proxy configuration
curl -v --proxy $PROXY_SERVER:$PROXY_PORT https://example.com
```

## Performance Profiling

Profile performance issues:

```bash
# CPU profiling
python diagnostic.py --profile cpu https://example.com

# Memory profiling
python diagnostic.py --profile memory https://example.com
```

## Common Debug Scenarios

### Screenshot Timing
```json
{
    "url": "https://example.com",
    "custom_js": "
        console.time('loadTime');
        await new Promise(resolve => {
            const observer = new MutationObserver(() => {
                if (document.readyState === 'complete') {
                    console.timeEnd('loadTime');
                    resolve();
                }
            });
            observer.observe(document, {subtree: true, childList: true});
        });
    "
}
```

### Memory Leaks
```bash
# Monitor memory over time
watch -n 5 'curl -s http://your-instance/health | jq .checks.memory_usage_mb'

# Check worker recycling
curl http://your-instance/health | jq '.checks.worker_stats'
```

## Debug Tools Reference

| Tool | Purpose | Usage |
|------|---------|-------|
| diagnostic.py | General troubleshooting | `python diagnostic.py <url>` |
| Health API | System monitoring | `curl /health` |
| Docker logs | Container logging | `docker logs pixashot` |
| Cloud Run logs | Production logging | `gcloud logging read` |

## Next Steps

1. Review [Common Issues](common-issues.md)
2. Check [FAQ](faq.md)
3. Contact support with debug output
4. Review system metrics