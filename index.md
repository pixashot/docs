---
title: Pixashot Documentation
excerpt: Complete documentation for Pixashot, a high-performance web screenshot service with extensive customization options for testing, archiving, and production use cases.
meta:
  nav_order: 10
---

# Pixashot Documentation

Welcome to Pixashot, a high-performance web screenshot service built for reliability and flexibility. Whether you're capturing screenshots for automated testing, content archival, or production applications, this documentation will help you make the most of Pixashot's capabilities.

## Quick Navigation

- 🚀 [Quick Start Guide](getting-started/quickstart.md) - Take your first screenshot in minutes
- 🎓 [Core Concepts](core-concepts/index.md) - Understand Pixashot's architecture
- 📚 [API Reference](api-reference/index.md) - Complete API documentation
- 🔧 [Troubleshooting](troubleshooting/index.md) - Solve common issues

## Learning Paths

### For First-Time Users
1. [Installation Guide](getting-started/installation.md)
2. [Quick Start Tutorial](getting-started/quickstart.md)
3. [Basic Configuration](getting-started/configuration.md)
4. [Common Use Cases](use-cases/index.md)

### For Developers
1. [Core Concepts](core-concepts/index.md)
2. [API Reference](api-reference/index.md)
3. [Client Libraries](code-examples/index.md)
4. [Advanced Features](capture-options/advanced-capture.md)

### For DevOps
1. [Deployment Overview](deployment/index.md)
2. [Security Guide](security/index.md)
3. [Resource Management](core-concepts/resource-management.md)
4. [Monitoring Setup](deployment/cloud-run.md)

## What is Pixashot?

Pixashot is a containerized web screenshot service that leverages modern browser automation for high-quality captures. Its unique single-context architecture ensures optimal performance and resource utilization, making it suitable for deployments of any scale.

### Key Features

#### Core Capabilities
- **High-Fidelity Capture**: Pixel-perfect screenshots with support for Retina displays
- **Multiple Formats**: PNG, JPEG, WebP, PDF, and HTML output
- **Flexible Viewports**: Custom sizes and pixel density
- **Full Page Support**: Intelligent scrollable content capture
- **Element Selection**: Target specific page elements

#### Advanced Features
- **Single Browser Context**: [Learn more](core-concepts/browser-context.md)
    - Efficient resource sharing
    - Consistent performance
    - Optimized memory usage
    - Reliable extension handling

- **Dynamic Content Handling**: [Learn more](interaction-system/dynamic-content.md)
    - Smart content waiting
    - Animation handling
    - Network state detection
    - Custom timing controls

- **Interaction Support**: [Learn more](interaction-system/user-actions.md)
    - Programmable clicks
    - Form interactions
    - Navigation sequences
    - Custom JavaScript execution

#### Performance & Security
- **Resource Management**: [Learn more](core-concepts/resource-management.md)
    - Smart memory handling
    - Automatic cleanup
    - Response caching
    - Rate limiting

- **Security Features**: [Learn more](security/index.md)
    - Token authentication
    - HTTPS support
    - Resource protection
    - Input validation

## Getting Started

### Quick Start with Docker
```bash
# Launch Pixashot
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  gpriday/pixashot:latest

# Take a screenshot
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true
  }'
```

### Production Deployment
For production environments, we recommend Google Cloud Run:
```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --memory 2Gi \
  --cpu 1 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=your_secret_token"
```

[Learn more about deployment options](deployment/index.md)

## System Requirements

| Environment | CPU | RAM | Storage |
|-------------|-----|-----|----------|
| Development | 1 core | 2GB | 1GB |
| Production | 2+ cores | 4GB+ | 5GB+ |
| High Traffic | 4+ cores | 8GB+ | 10GB+ |

[Detailed requirements guide](getting-started/installation.md#system-requirements)

## Documentation Structure

### Core Documentation
- **[Getting Started](getting-started/index.md)**
    - Installation and setup
    - Basic configuration
    - Quick start guide
    - Best practices

- **[Core Concepts](core-concepts/index.md)**
    - Architecture overview
    - Browser context
    - Request lifecycle
    - Resource management

- **[Capture Options](capture-options/index.md)**
    - Basic captures
    - Advanced features
    - Output formats
    - Viewport settings

### Advanced Topics
- **[Interaction System](interaction-system/index.md)**
    - User actions
    - Dynamic content
    - Wait strategies

- **[Deployment](deployment/index.md)**
    - Platform guides
    - Scaling strategies
    - Monitoring setup

- **[Security](security/index.md)**
    - Authentication
    - Network security
    - Best practices

### Reference & Support
- **[API Reference](api-reference/index.md)**
    - Endpoints
    - Request options
    - Response handling
    - Rate limiting

- **[Code Examples](code-examples/index.md)**
    - Python
    - Node.js
    - PHP
    - Other languages

- **[Use Cases](use-cases/index.md)**
    - E-commerce
    - Testing
    - Archiving

- **[Troubleshooting](troubleshooting/index.md)**
    - Common issues
    - Debugging guide
    - FAQ

## Cost Efficiency

Most users stay within Google Cloud Run's free tier:
- 2 million requests/month included
- 360,000 vCPU-seconds
- 180,000 GiB-seconds

[Learn more about pricing and optimization](deployment/index.md#cost-considerations)

## Support & Community

- **[GitHub Issues](https://github.com/pixashot/pixashot/issues)**: Bug reports and feature requests
- **[GitHub Discussions](https://github.com/pixashot/pixashot/discussions)**: Community discussions
- **[Email Support](mailto:support@pixashot.com)**: Direct assistance
- **[Stack Overflow](https://stackoverflow.com/questions/tagged/pixashot)**: Community Q&A

## License & Acknowledgements

Pixashot is open source software licensed under the MIT license and built with:
- [Playwright](https://playwright.dev/) - Browser automation
- [Quart](https://quart.palletsprojects.com/) - Async web framework
- [PopUpOFF](https://github.com/AdguardTeam/PopUpOFF) - Popup blocking
- [I don't care about cookies](https://www.i-dont-care-about-cookies.eu/) - Cookie handling

[View License](https://github.com/pixashot/pixashot/blob/main/LICENSE)