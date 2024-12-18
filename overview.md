---
title: Overview of Pixashot
excerpt: A comprehensive introduction to Pixashot's capabilities, features, deployment options, and documentation
structure for screenshot automation and web content capture.
meta:
  nav_order: 0
---

# Pixashot Overview

## What is Pixashot?

Pixashot is a high-performance web screenshot service designed to convert web pages and HTML content into high-quality
images and PDFs. Built for reliability, scalability, and ease of use, it leverages modern browser automation and a
unique single-context architecture to deliver consistent, efficient screenshots at any scale.

Whether you need screenshots for automated testing, content archiving, visual regression testing, or dynamic content
generation for production applications, Pixashot offers the customization, performance, and security features to meet
your needs.

## Core Features

### Screenshot Capabilities

- **High-Fidelity Capture**: Generate pixel-perfect screenshots, including support for Retina displays and custom
  resolutions.
- **Multiple Output Formats**: Save screenshots as PNG, JPEG, WebP, PDF, or even capture the full HTML content.
- **Flexible Viewports**: Customize viewport sizes and set device pixel ratios to simulate any device.
- **Full Page Screenshots**: Intelligently capture entire scrollable web pages, not just the visible area.
- **Element Selection**: Target specific elements on a page using CSS selectors for precise captures.

### Advanced Features

- **Single Browser Context**: Optimizes performance through efficient resource sharing, reduced memory footprint, and
  faster capture speeds. ([Learn more](core-concepts/browser-context.md))
- **Dynamic Content Handling**: Effectively handles JavaScript-rendered content, complex animations, and single-page
  applications (SPAs). ([Learn more](interaction-system/dynamic-content.md))
- **Interactive Capture**: Automate user interactions like clicks, form submissions, typing, and scrolling before
  capturing screenshots. ([Learn more](interaction-system/user-actions.md))
- **Smart Wait Strategies**: Intelligently wait for network events, elements, or custom conditions before
  capturing. ([Learn more](interaction-system/wait-strategies.md))
- **Resource Management**: Offers automatic resource cleanup, response caching, and configurable rate
  limiting. ([Learn more](core-concepts/resource-management.md))
- **Security**: Supports token-based authentication, signed URLs, HTTPS, and comprehensive security best
  practices. ([Learn more](security/index.md))
- **Customization**: Execute custom JavaScript, block specific network requests, set custom headers, and simulate
  various network conditions. ([Learn more](capture-options/advanced-capture.md))

## Getting Started

### Quick Start with Docker

```bash
# Launch Pixashot with Docker
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
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
  --region us-central1 \
  --memory 2Gi \
  --cpu 1 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=your_secret_token"
```

[See all deployment options](deployment/index.md)

## System Requirements

| Environment  | CPU      | RAM  | Storage |
|--------------|----------|------|---------|
| Development  | 1 core   | 2GB  | 1GB     |
| Production   | 2+ cores | 4GB+ | 5GB+    |
| High Traffic | 4+ cores | 8GB+ | 10GB+   |

[View detailed requirements](getting-started/installation.md#system-requirements)

## Documentation Guide

### Getting Started

1. [Installation Guide](getting-started/installation.md) - Set up Pixashot using Docker or deploy to cloud platforms
   like Google Cloud Run.
2. [Quick Start Tutorial](getting-started/quickstart.md) - Capture your first screenshots and learn basic usage.
3. [Configuration Guide](getting-started/configuration.md) - Configure Pixashot using environment variables and
   understand available settings.
4. [Common Use Cases](use-cases/index.md) - Explore practical applications and implementation patterns.

### Developer Resources

1. [Core Concepts](core-concepts/index.md) - Understand Pixashot's architecture, including the single browser context,
   request lifecycle, and resource management.
2. [API Reference](api-reference/index.md) - Comprehensive documentation of all API endpoints, request parameters, and
   response formats.
3. [Code Examples](code-examples/index.md) - Language-specific examples (Python, Node.js, PHP) demonstrating common
   usage patterns.
4. [Advanced Features](capture-options/advanced-capture.md) - Deep dive into advanced capture options like dark mode,
   geolocation, and custom JavaScript.

### Operations Guide

1. [Deployment Guide](deployment/index.md) - Detailed instructions for various deployment platforms, including scaling
   considerations.
2. [Security Guide](security/index.md) - Learn about authentication, network security, and best practices for secure
   deployments.
3. [Resource Management](core-concepts/resource-management.md) - Best practices for optimizing resource usage, including
   memory, CPU, and browser resources.
4. [Monitoring Guide](deployment/cloud-run.md) - Set up monitoring, logging, and alerting for production deployments.

## Cost Efficiency

Pixashot is designed for cost-effectiveness. Many users can operate within Google Cloud Run's free tier, which includes:

- 2 million requests per month
- 360,000 vCPU-seconds
- 180,000 GiB-seconds

[Learn about cost optimization](deployment/index.md#cost-optimization-strategies)

## Technology Stack

Pixashot is built using:

- [Playwright](https://playwright.dev/) - A powerful Node.js library for automating Chromium, Firefox, and WebKit.
- [Quart](https://quart.palletsprojects.com/) - An asynchronous Python web framework based on the Flask API.
- [PopUpOFF](https://github.com/AdguardTeam/PopUpOFF) - A browser extension for blocking popups.
- [I don't care about cookies](https://www.i-dont-care-about-cookies.eu/) - A browser extension for handling cookie
  consent notices.

## Support & Community

- **Issues & Bugs**: [GitHub Issues](https://github.com/pixashot/pixashot/issues)
- **Discussions**: [GitHub Discussions](https://github.com/pixashot/pixashot/discussions)
- **Direct Support**: [Email Support](mailto:support@pixashot.com)
- **Q&A**: [Stack Overflow](https://stackoverflow.com/questions/tagged/pixashot)

## License

Pixashot is open-source software, licensed under the MIT license.

[View License](https://github.com/pixashot/pixashot/blob/main/LICENSE)