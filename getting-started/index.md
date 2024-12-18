---
title: Getting Started with Pixashot
excerpt: A comprehensive introduction to Pixashot, including system requirements, prerequisites, core concepts, and
initial setup guidance.
meta:
  nav_order: 20
---

# Getting Started with Pixashot

Welcome to Pixashot! This guide will help you get up and running with our high-performance screenshot service. Whether you're setting up a development environment or deploying to production, this section will guide you through the process.

## What You'll Need

Before getting started with Pixashot, ensure you have the following:

### System Requirements

* **CPU**: 1 core minimum (2+ cores recommended for production)
* **Memory**: 2GB minimum (4GB+ recommended for production)
* **Storage**: 1GB available space
* **Operating System**: Any Docker-compatible OS (Linux recommended)
* **Network**: Outbound internet access
* **Docker**: Version 20.10.0 or higher (if using Docker deployment)

### For Cloud Run Deployment

* Google Cloud SDK installed
* Google Cloud project with billing enabled
* Required APIs enabled:
    * Cloud Run API
    * Artifact Registry API (for storing images if using Cloud Build)
    * Secret Manager API (if using secrets for authentication tokens)

### Development Tools

* `curl` or similar HTTP client for testing
* Text editor for configuration files
* Basic command line knowledge

## Core Concepts

Before diving into installation, it's helpful to understand these key concepts:

### Single Browser Context

Pixashot uses a single browser context architecture where:

* One browser instance serves all requests
* Resources are efficiently shared
* Extensions are configured globally
* Memory usage is optimized

This approach provides significant performance benefits compared to traditional methods.

### Authentication

Pixashot supports token-based authentication, which is highly recommended for production environments. You can set up
authentication using environment variables:

```bash
# Example authentication token setup
export AUTH_TOKEN=$(openssl rand -hex 32)
```

### Request Flow

A basic screenshot request follows this pattern:

1. **Authentication validation**: The service checks for a valid authentication token or signed URL.
2. **Parameter processing**: Request parameters are validated and processed.
3. **Browser interaction**: A new page is created in the shared browser context, and interactions are executed.
4. **Screenshot capture**: The screenshot is taken based on the provided options.
5. **Response delivery**: The captured image or PDF is returned to the client.

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

This should return a PNG image of the example.com homepage.

## What's Next?

Follow these steps to get started:

1. **[Installation](installation.md)**
    * Docker setup
    * Cloud Run deployment
    * Health check verification

2. **[Quick Start Guide](quickstart.md)**
    * First screenshot
    * Basic customization
    * Error handling

3. **[Configuration](configuration.md)**
    * Environment variables
    * Template setup
    * Rate limiting

4. **[Best Practices](best-practices.md)**
    * Resource optimization
    * Error handling
    * Performance tuning

## Common Questions

### How much will it cost?

Most users can operate within Google Cloud Run's free tier:

* 2 million requests/month included
* 360,000 vCPU-seconds
* 180,000 GiB-seconds

For detailed pricing, refer to the [Google Cloud Run pricing page](https://cloud.google.com/run/pricing).

### What browsers are supported?

Pixashot uses Chromium through Playwright, offering:

* Modern web standards support
* Consistent rendering across platforms
* Extensive browser automation features

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

* Adjust worker count (`WORKERS`)
* Configure memory limits
* Set appropriate rate limits
* Enable response caching

## Resource Checklist

Before proceeding with installation, ensure you have:

*   [ ] Chosen deployment platform (Docker or Cloud Run)
*   [ ] Met minimum system requirements
*   [ ] Generated a secure authentication token
*   [ ] Prepared environment variables
*   [ ] Enabled required cloud APIs (if using Cloud Run)
*   [ ] Basic understanding of screenshot requirements

## Available Features

Pixashot supports a wide range of capture options:

* **Formats**: PNG, JPEG, WebP, PDF, HTML
* **Capture Types**: Full page, element specific, viewport specific
* **Customization**: Custom viewports, pixel density, custom headers
* **Interactions**: Click, type, wait, scroll actions
* **Output**: Direct binary, base64 encoding, or saved files

## Getting Help

If you encounter issues:

1. Check the [troubleshooting guide](../troubleshooting/index.md)
2. Review [common issues](../troubleshooting/common-issues.md)
3. Search [GitHub Issues](https://github.com/gregpriday/pixashot/issues)
4. Contact [support@pixashot.com](mailto:support@pixashot.com)

## Security Note

While getting started, remember to:

* Keep authentication tokens secure
* Use HTTPS in production
* Configure appropriate rate limits
* Follow security best practices from our [security guide](../security/index.md)

## Next Steps

Ready to begin? Head to the [Installation Guide](installation.md) to set up your Pixashot instance.