---
title: Troubleshooting Guide
excerpt: Overview of common problems and solutions when using Pixashot, including diagnostic tools and debugging procedures.
meta:
    nav_order: 170
---

# Troubleshooting Pixashot

This guide helps you diagnose and resolve common issues with Pixashot. Follow our structured approach to quickly identify and fix problems.

## Quick Diagnostic Steps

1. **Check Health Endpoints**
```bash
# Basic health check
curl http://your-instance/health

# Readiness check
curl http://your-instance/health/ready

# Liveness check
curl http://your-instance/health/live
```

2. **Use Diagnostic Script**
```bash
# Run diagnostic tool on problematic URL
python diagnostic.py https://example.com
```

3. **Check Logs**
```bash
# Docker logs
docker logs pixashot

# Cloud Run logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=pixashot"
```

## Common Error Types

### Network Issues
- Timeouts
- SSL/TLS errors
- DNS resolution failures
- Proxy configuration problems

### Browser Context Issues
- Memory exhaustion
- Worker process crashes
- Extension failures
- Page load timeouts

### Resource Issues
- Out of memory errors
- High CPU usage
- Disk space limitations
- Network bandwidth constraints

## First Response Checklist

1. **Verify Configuration**
    - Check environment variables
    - Confirm authentication settings
    - Validate proxy configuration
    - Review resource limits

2. **Monitor Resources**
    - Memory usage
    - CPU utilization
    - Network activity
    - Disk space

3. **Check Dependencies**
    - Browser version
    - Extension status
    - Network connectivity
    - System resources

## Getting Help

1. **Review Documentation**
    - [Common Issues](common-issues.md)
    - [Debugging Guide](debugging.md)
    - [FAQ](faq.md)

2. **Support Channels**
    - GitHub Issues
    - Email Support
    - Community Forums
    - Documentation

## Next Steps

- Review [Common Issues](common-issues.md) for specific problems
- Learn about [Debugging Tools](debugging.md)
- Check [FAQ](faq.md) for quick answers
- Follow [Best Practices](../getting-started/best-practices.md)