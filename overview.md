---
title: Overview of Pixashot
excerpt: A comprehensive introduction to Pixashot's capabilities, features, and documentation structure for screenshot automation and web content capture.
meta:
  nav_order: 0
---

# Pixashot Overview

## What is Pixashot?

Pixashot is a high-performance web screenshot service that converts web pages into high-quality images and PDFs. Built for reliability and flexibility, it uses modern browser automation and a unique single-context architecture to deliver consistent, efficient screenshots at any scale.

Whether you need screenshots for automated testing, content archiving, or production applications, Pixashot offers the customization and performance to meet your needs.

## Core Features

### Screenshot Capabilities
- **High-Fidelity Capture**: Generate pixel-perfect screenshots, including Retina display support
- **Multiple Output Formats**: Save as PNG, JPEG, WebP, PDF, or HTML
- **Flexible Viewports**: Customize sizes and pixel density for any device
- **Full Page Support**: Intelligently capture entire scrollable content
- **Element Selection**: Target specific page elements with precision

### Advanced Features
- **Single Browser Context**: Optimize performance through efficient resource sharing and memory management ([Learn more](core-concepts/browser-context.md))
- **Dynamic Content**: Handle JavaScript-rendered content, animations, and complex web applications ([Learn more](interaction-system/dynamic-content.md))
- **Interactive Capture**: Automate clicks, form inputs, and navigation before capturing ([Learn more](interaction-system/user-actions.md))
- **Smart Resource Management**: Auto-cleanup, caching, and rate limiting ([Learn more](core-concepts/resource-management.md))
- **Enterprise Security**: Token authentication, HTTPS, and comprehensive resource protection ([Learn more](security/index.md))

## Getting Started

### Quick Start with Docker
```bash
# Launch Pixashot
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  gpriday/pixashot:latest

# Take your first screenshot
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true
  }'
```

### Production on Google Cloud Run
```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --memory 2Gi \
  --cpu 1 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=your_secret_token"
```

[See all deployment options](deployment/index.md)

## System Requirements

| Environment | CPU | RAM | Storage |
|-------------|-----|-----|----------|
| Development | 1 core | 2GB | 1GB |
| Production | 2+ cores | 4GB+ | 5GB+ |
| High Traffic | 4+ cores | 8GB+ | 10GB+ |

[View detailed requirements](getting-started/installation.md#system-requirements)

## Documentation Guide

### Getting Started
1. [Installation Guide](getting-started/installation.md) - Set up Pixashot
2. [Quick Start Tutorial](getting-started/quickstart.md) - Take your first screenshots
3. [Configuration Guide](getting-started/configuration.md) - Configure your installation
4. [Common Use Cases](use-cases/index.md) - See practical applications

### Developer Resources
1. [Core Concepts](core-concepts/index.md) - Understand the architecture
2. [API Reference](api-reference/index.md) - Complete API documentation
3. [Code Examples](code-examples/index.md) - Language-specific examples
4. [Advanced Features](capture-options/advanced-capture.md) - Advanced usage

### Operations Guide
1. [Deployment Guide](deployment/index.md) - Production deployment
2. [Security Guide](security/index.md) - Security configuration
3. [Resource Management](core-concepts/resource-management.md) - Resource optimization
4. [Monitoring Guide](deployment/cloud-run.md) - Production monitoring

## Cost Efficiency

Pixashot is designed to be cost-effective. Most users operate within Google Cloud Run's free tier:
- 2 million requests/month included
- 360,000 vCPU-seconds
- 180,000 GiB-seconds

[Learn about cost optimization](deployment/index.md#cost-considerations)

## Technology Stack

Pixashot is built on proven, reliable technologies:
- [Playwright](https://playwright.dev/) - Modern browser automation
- [Quart](https://quart.palletsprojects.com/) - Async web framework
- [PopUpOFF](https://github.com/AdguardTeam/PopUpOFF) - Popup handling
- [I don't care about cookies](https://www.i-dont-care-about-cookies.eu/) - Cookie management

## Support & Community

- **Issues & Bugs**: [GitHub Issues](https://github.com/pixashot/pixashot/issues)
- **Discussions**: [GitHub Discussions](https://github.com/pixashot/pixashot/discussions)
- **Direct Support**: [Email Support](mailto:support@pixashot.com)
- **Q&A**: [Stack Overflow](https://stackoverflow.com/questions/tagged/pixashot)

## License

Pixashot is open source software, licensed under the MIT license.

[View License](https://github.com/pixashot/pixashot/blob/main/LICENSE)