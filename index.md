# Pixashot Documentation

Welcome to Pixashot, a high-performance web screenshot service built for reliability and flexibility. Pixashot provides pixel-perfect screenshot capture with extensive customization options, making it ideal for automated testing, content archival, and various production use cases.

## What is Pixashot?

Pixashot is a containerized web screenshot service that uses modern browser automation to capture high-quality screenshots. It's designed with a single-context architecture for optimal performance and resource utilization, making it suitable for both small-scale deployments and high-traffic production environments.

### Key Features

#### Core Capabilities
- **High-Fidelity Capture**: Pixel-perfect screenshots at any resolution, including Retina displays
- **Multiple Formats**: Support for PNG, JPEG, WebP, PDF, and HTML output
- **Flexible Viewports**: Custom viewport sizes with configurable pixel density
- **Full Page Support**: Intelligent capture of scrollable content with dynamic height detection
- **Element Selection**: Capture specific page elements using CSS selectors

#### Advanced Features
- **Single Browser Context**: Efficient resource sharing and consistent performance
- **Dark Mode Support**: Accurate dark mode simulation and capture
- **Geolocation Spoofing**: Precise location simulation for testing
- **Custom JavaScript**: Inject and execute custom JavaScript before capture
- **Interaction Sequences**: Programmable clicks, typing, and navigation actions
- **Dynamic Content**: Smart waiting for dynamic content and animations

#### Performance & Reliability
- **Resource Optimization**: Smart browser instance pooling and memory management
- **Response Caching**: Optional caching for repeated captures (configurable)
- **Rate Limiting**: Configurable request throttling
- **Health Monitoring**: Built-in health checks and performance metrics
- **Error Recovery**: Automatic cleanup and recovery from failed captures

#### Security
- **Authentication**: Token-based authentication and signed URLs
- **Network Security**: HTTPS support and proxy configuration
- **Resource Protection**: Memory and CPU limits with automatic cleanup
- **Input Validation**: Comprehensive request validation and sanitization

## Quick Start

The fastest way to get started with Pixashot is using Docker:

```bash
# Launch Pixashot
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

## Production Deployment

For production deployments, we recommend Google Cloud Run:

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

Cloud Run provides:
- Automatic scaling
- Zero management overhead
- Pay-per-use pricing
- Global deployment options
- Built-in monitoring and logging

## Common Use Cases

Pixashot is used across various industries for:

- **Visual Testing**: Automated UI validation and regression testing
- **Content Archiving**: High-fidelity web page preservation
- **E-commerce**: Product image and listing capture
- **Social Media**: Dynamic preview generation
- **Documentation**: Technical documentation screenshots
- **SEO Monitoring**: Visual change tracking
- **Legal Compliance**: Document preservation and verification
- **Content Moderation**: Visual content review automation

## System Requirements

Minimum requirements for running Pixashot:
- CPU: 1 core (2+ recommended for production)
- RAM: 2GB minimum (4GB+ recommended for production)
- Storage: 1GB for installation
- Operating System: Any Docker-compatible OS

## Cost Efficiency

When deployed on Google Cloud Run, most users stay within the free tier:
- 2 million requests/month included
- 360,000 vCPU-seconds
- 180,000 GiB-seconds
- Pay only for what you use beyond free tier

## Documentation Overview

Our documentation is organized into the following sections:

- [Getting Started](getting-started/): Installation, configuration, and basic usage
- [Core Concepts](core-concepts/): Architecture, browser context, and request lifecycle
- [Capture Options](capture-options/): Screenshot customization and output formats
- [Interaction System](interaction-system/): Page interactions and dynamic content handling
- [Deployment](deployment/): Deployment guides and scaling considerations
- [Security](security/): Authentication, network security, and best practices
- [API Reference](api-reference/): Complete API documentation
- [Client Libraries](client-libraries/): Official and community client implementations
- [Use Cases](use-cases/): Common implementation patterns
- [Troubleshooting](troubleshooting/): Common issues and solutions

## Support and Community

- **Documentation**: You're here! Browse the sections above for detailed information
- **Issue Tracking**: Report issues on [GitHub Issues](https://github.com/pixashot/pixashot/issues)
- **Feature Requests**: Discuss new features on [GitHub Discussions](https://github.com/pixashot/pixashot/discussions)
- **Email Support**: Contact us at [support@pixashot.com](mailto:support@pixashot.com)

## Acknowledgements

Pixashot is built with powerful open source technologies:
- [Playwright](https://playwright.dev/) for reliable browser automation
- [Quart](https://quart.palletsprojects.com/) for async web framework
- [PopUpOFF](https://github.com/AdguardTeam/PopUpOFF) for popup blocking
- [I don't care about cookies](https://www.i-dont-care-about-cookies.eu/) for cookie notice handling

## License

Pixashot is open source software licensed under the MIT license. See the [LICENSE](https://github.com/pixashot/pixashot/blob/main/LICENSE) file for more details.