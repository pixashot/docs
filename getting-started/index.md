---
title: Getting Started with Pixashot
excerpt: A comprehensive introduction to Pixashot, including system requirements, prerequisites, and initial setup guidance.
meta:
    nav_order: 20
---

# Getting Started with Pixashot

Welcome to Pixashot! This guide will help you get up and running with our high-performance screenshot service. Whether you're setting up a development environment or deploying to production, this section will guide you through the process.

## What You'll Need

Before getting started with Pixashot, ensure you have:

### System Requirements

- **CPU**: 1 core minimum (2+ cores recommended for production)
- **Memory**: 2GB minimum (4GB+ recommended for production)
- **Storage**: 1GB available space
- **Operating System**: Any Docker-compatible OS (Linux recommended)
- **Network**: Outbound internet access
- **Docker**: Version 20.10.0 or higher (if using Docker deployment)

### For Cloud Run Deployment

- Google Cloud SDK installed
- Google Cloud project with billing enabled
- Required APIs enabled:
    - Cloud Run API
    - Container Registry API
    - Secret Manager API (if using secrets)

### Development Tools

- `curl` or similar HTTP client for testing
- Text editor for configuration files
- Basic command line knowledge

## Core Concepts

Before diving into installation, it's helpful to understand these key concepts:

### Single Browser Context

Pixashot uses a single browser context architecture where:
- One browser instance serves all requests
- Resources are efficiently shared
- Extensions are configured globally
- Memory usage is optimized

### Authentication

Every Pixashot can have authentication, which is ideal for production:
```bash
# Example authentication token setup
export AUTH_TOKEN=$(openssl rand -hex 32)
```

### Request Flow

A basic screenshot request follows this pattern:
1. Authentication validation
2. Parameter processing
3. Browser interaction
4. Screenshot capture
5. Response delivery

## Quick Verification

After installation, you can verify your setup with this simple request:

```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png"
  }'
```

## What's Next?

Follow these steps to get started:

1. **[Installation](installation.md)**
    - Docker setup
    - Cloud Run deployment
    - Health check verification

2. **[Quick Start Guide](quickstart.md)**
    - First screenshot
    - Basic customization
    - Error handling

3. **[Configuration](configuration.md)**
    - Environment variables
    - Template setup
    - Rate limiting

4. **[Best Practices](best-practices.md)**
    - Resource optimization
    - Error handling
    - Performance tuning

## Common Questions

### How much will it cost?
Most users stay within Google Cloud Run's free tier:
- 2 million requests/month
- 360,000 vCPU-seconds
- 180,000 GiB-seconds

### What browsers are supported?
Pixashot uses Chromium through Playwright, offering:
- Modern web standards support
- Consistent rendering
- Extensive feature support

### Can I run Pixashot behind a proxy?
Yes, Pixashot supports proxy configuration through environment variables:
```bash
export PROXY_SERVER="proxy.example.com"
export PROXY_PORT="8080"
export PROXY_USERNAME="user"
export PROXY_PASSWORD="pass"
```

### How do I handle high traffic?
Pixashot scales automatically on Cloud Run. For other deployments:
- Adjust worker count (`WORKERS`)
- Configure memory limits
- Set appropriate rate limits
- Enable response caching

## Resource Checklist

Before proceeding with installation, ensure you have:

- [ ] Chosen deployment platform (Docker or Cloud Run)
- [ ] Met minimum system requirements
- [ ] Generated secure authentication token
- [ ] Prepared environment variables
- [ ] Enabled required cloud APIs (if using Cloud Run)
- [ ] Basic understanding of screenshot requirements

## Available Features

Pixashot supports various capture options:

- **Formats**: PNG, JPEG, WebP, PDF
- **Capture Types**: Full page, element specific, viewport
- **Customization**: Custom viewports, pixel density, headers
- **Interactions**: Click, type, wait, scroll actions
- **Output**: Direct binary, base64 encoding, or saved files

## Getting Help

If you encounter issues:

1. Check the [troubleshooting guide](../troubleshooting/index.md)
2. Review [common issues](../troubleshooting/common-issues.md)
3. Search [GitHub Issues](https://github.com/pixashot/pixashot/issues)
4. Contact [support@pixashot.com](mailto:support@pixashot.com)

## Security Note

While getting started, remember to:
- Keep authentication tokens secure
- Use HTTPS in production
- Configure appropriate rate limits
- Follow security best practices from our [security guide](../security/index.md)

## Next Steps

Ready to begin? Head to the [Installation Guide](installation.md) to set up your Pixashot instance.