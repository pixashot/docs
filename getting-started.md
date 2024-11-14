# Getting Started

## Introduction

Pixashot is a powerful, flexible, and developer-friendly web screenshot service that simplifies the process of capturing high-quality screenshots. Whether you need to automate web testing, generate thumbnails, or create visual archives of web pages, Pixashot provides a robust solution with its easy-to-use API.

Key features of Pixashot include:

- High-quality screenshots at any resolution, including Retina displays
- Full-page captures and element-specific screenshots
- Multiple output formats (PNG, JPEG, WebP, PDF, HTML)
- Structured interaction sequences
- Device simulation with user agent customization
- Dark mode support and animation handling
- Built-in popup and cookie consent blockers
- Template system for common scenarios
- Comprehensive PDF generation options
- Flexible deployment options (Cloud Run, Docker, or self-hosted)

## Quick Start with Docker

The fastest way to get started with Pixashot is using our pre-built Docker image:

```bash
docker pull gpriday/pixashot:latest
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest
```

Once running, you can test it with a simple API call:

```javascript
const axios = require('axios');
const fs = require('fs');

const PIXASHOT_API_URL = 'http://localhost:8080/capture';
const AUTH_TOKEN = 'your_secret_token';

axios.post(PIXASHOT_API_URL, {
  url: 'https://example.com',
  format: 'png',
  full_page: true,
  wait_for_animation: true,
  template: 'desktop',  // Use predefined template
  interactions: [
    {
      action: 'wait_for',
      wait_for: {
        type: 'network_idle',
        value: 2000
      }
    }
  ]
}, {
  headers: {
    'Authorization': `Bearer ${AUTH_TOKEN}`,
    'Content-Type': 'application/json'
  },
  responseType: 'arraybuffer'
})
.then(response => {
  fs.writeFileSync('screenshot.png', response.data);
  console.log('Screenshot saved as screenshot.png');
})
.catch(error => {
  console.error('Error capturing screenshot:', error.message);
});
```

## Basic Examples

### Capture Full Page
```javascript
{
  "url": "https://example.com",
  "format": "png",
  "full_page": true,
  "wait_for_animation": true
}
```

### Capture Mobile View
```javascript
{
  "url": "https://example.com",
  "template": "mobile",
  "format": "png",
  "dark_mode": true,
  "pixel_density": 2.0
}
```

### Capture with Interactions
```javascript
{
  "url": "https://example.com",
  "format": "png",
  "interactions": [
    {
      "action": "click",
      "selector": "#accept-cookies"
    },
    {
      "action": "type",
      "selector": "#search",
      "text": "example"
    },
    {
      "action": "wait_for",
      "wait_for": {
        "type": "network_idle",
        "value": 2000
      }
    }
  ]
}
```

## Configuration

### Required Environment Variables
- `AUTH_TOKEN`: Your authentication token for API requests
- `PORT`: Server port (default: 8080)

### Browser Extension Settings
- `USE_POPUP_BLOCKER`: Enable/disable popup blocker (default: true)
- `USE_COOKIE_BLOCKER`: Enable/disable cookie consent blocker (default: true)

### Worker Configuration
- `WORKERS`: Number of worker processes (default: 4)
- `MAX_REQUESTS`: Maximum requests per worker (default: 1000)
- `KEEP_ALIVE`: Keep-alive timeout in seconds (default: 300)

### Optional Settings
- `RATE_LIMIT_ENABLED`: Enable rate limiting (default: false)
- `RATE_LIMIT_CAPTURE`: Rate limit for capture endpoint (default: "1 per second")
- `RATE_LIMIT_SIGNED`: Rate limit for signed URLs (default: "5 per second")
- `CACHE_MAX_SIZE`: Maximum size for response caching (default: 0, disabled)

## Installation Methods

### Docker (Recommended)

```bash
# Basic setup
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest

# Advanced setup with all options
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  -e WORKERS=4 \
  -e MAX_REQUESTS=1000 \
  -e KEEP_ALIVE=300 \
  -e RATE_LIMIT_ENABLED=true \
  -e RATE_LIMIT_CAPTURE="1 per second" \
  -e RATE_LIMIT_SIGNED="5 per second" \
  -e CACHE_MAX_SIZE=1000 \
  --memory=2g \
  gpriday/pixashot:latest
```

### Google Cloud Run

For cloud deployments, we recommend Google Cloud Run. Basic deployment:

```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 2Gi \
  --cpu 2 \
  --port 8080 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=your_secret_token,\
USE_POPUP_BLOCKER=true,\
USE_COOKIE_BLOCKER=true,\
WORKERS=4,\
MAX_REQUESTS=1000,\
KEEP_ALIVE=300"
```

See our [Cloud Run Deployment Guide](deployment-cloud-run.md) for detailed instructions.

## Authentication

Pixashot supports two authentication methods:

### 1. Bearer Token Authentication

Include the `AUTH_TOKEN` in the `Authorization` header:

```http
Authorization: Bearer your_secret_token
```

### 2. Signed URLs

For scenarios where you can't include headers:

```javascript
const crypto = require('crypto');

function generateSignedUrl(baseUrl, params, secretKey, expiresIn = 3600) {
  const expires = Math.floor(Date.now() / 1000) + expiresIn;
  const signature = crypto.createHmac('sha256', secretKey)
    .update(`${baseUrl}?${new URLSearchParams(params)}&expires=${expires}`)
    .digest('hex');
  
  return `${baseUrl}?${new URLSearchParams(params)}&expires=${expires}&signature=${signature}`;
}
```

## Health Checks

Pixashot provides three health check endpoints:

```bash
# Basic liveness check
curl http://localhost:8080/health/live

# Readiness check
curl http://localhost:8080/health/ready

# Detailed health status
curl http://localhost:8080/health
```

## Templates

Pixashot includes built-in templates for common scenarios:

```javascript
// Mobile device capture
{
  "url": "https://example.com",
  "template": "mobile",
  "format": "png"
}

// Desktop full page
{
  "url": "https://example.com",
  "template": "desktop_full",
  "format": "png"
}

// PDF article
{
  "url": "https://example.com",
  "template": "article_pdf",
  "format": "pdf"
}
```

## Best Practices

1. **Resource Management**
   - Start with 2GB memory minimum
   - Monitor worker process health
   - Use templates for consistent captures

2. **Performance Optimization**
   - Use appropriate wait strategies
   - Enable caching for repeated requests
   - Set reasonable timeouts
   - Use structured interactions

3. **Quality Control**
   - Set appropriate pixel density
   - Configure viewport sizes correctly
   - Use wait_for_animation when needed
   - Handle dark mode appropriately

4. **Error Handling**
   - Implement retry logic
   - Handle timeouts gracefully
   - Monitor error rates
   - Use health checks

Remember that Pixashot uses a single browser context for all requests, which means:
- Extension settings are server-wide
- Resources are shared efficiently
- Browser configuration is consistent
- Performance is optimized

For more advanced features and configurations, see our:
- [Advanced Usage Guide](advanced-usage.md)
- [API Reference](api-reference.md)
- [Deployment Guide](deployment.md)
- [Example Use Cases](example-use-cases.md)