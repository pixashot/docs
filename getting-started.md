---
excerpt: "Quick start guide for Pixashot, covering installation, authentication, and basic usage of the web screenshot service API."
published_at: true
---

# Getting Started

## Introduction

Pixashot is a powerful, flexible, and developer-friendly web screenshot service that simplifies the process of capturing high-quality screenshots. Whether you need to automate web testing, generate thumbnails, or create visual archives of web pages, Pixashot provides a robust solution with its easy-to-use API.

Key features of Pixashot include:

- High-quality screenshots at any resolution, including Retina displays
- Full-page captures and custom viewport sizes
- Multiple output formats (PNG, JPEG, WebP, PDF)
- Custom JavaScript injection for page manipulation
- Built-in popup and cookie consent blockers (configurable via environment variables)
- Flexible deployment options (Cloud Run, Docker, or self-hosted)
- Single browser context for efficient resource usage
- Configurable rate limiting and caching

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
  wait_for_network: 'idle'
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
  -e CACHE_MAX_SIZE=1000 \
  --memory=2g \
  gpriday/pixashot:latest
```

### Google Cloud Run

For cloud deployments, we recommend Google Cloud Run. See our [Cloud Run Deployment Guide](deployment-cloud-run.md) for detailed instructions.

Basic deployment:
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

## Authentication

Pixashot uses token-based authentication to secure API access. You need to include your authentication token in one of two ways:

### 1. Bearer Token Authentication

Include the `AUTH_TOKEN` in the `Authorization` header:

```http
Authorization: Bearer your_secret_token
```

### 2. Signed URLs

For scenarios where you can't include headers, use signed URLs:

```javascript
const crypto = require('crypto');

function generateSignedUrl(baseUrl, params, secretKey, expiresIn = 3600) {
  const expires = Math.floor(Date.now() / 1000) + expiresIn;
  const signature = crypto.createHmac('sha256', secretKey)
    .update(`${baseUrl}?${new URLSearchParams(params)}&expires=${expires}`)
    .digest('hex');
  
  return `${baseUrl}?${new URLSearchParams(params)}&expires=${expires}&signature=${signature}`;
}

const signedUrl = generateSignedUrl(
  'http://your-pixashot-instance.com/capture',
  { url: 'https://example.com', format: 'png' },
  'your_secret_key'
);
```

## Health Checks

Pixashot provides three health check endpoints:

```bash
# Basic liveness check
curl http://localhost:8080/health/live

# Readiness check (including browser status)
curl http://localhost:8080/health/ready

# Detailed health status
curl http://localhost:8080/health
```

## Best Practices

1. **Resource Management**
    - Start with 2GB memory minimum
    - Monitor worker process health
    - Use `wait_for_network` appropriately

2. **Performance Optimization**
    - Enable caching for repeated requests
    - Use `block_media: true` when images aren't needed
    - Set appropriate viewport sizes

3. **Security**
    - Keep your AUTH_TOKEN secure
    - Use HTTPS in production
    - Implement rate limiting for public instances

4. **Error Handling**
    - Implement retry logic with exponential backoff
    - Handle timeouts gracefully
    - Monitor error rates

Remember that Pixashot uses a single browser context for all requests, which means:
- Extension settings (popup/cookie blocking) are server-wide
- Better resource utilization
- More consistent performance
- Shared browser configuration across requests

For more advanced features and configurations, see our [Advanced Usage Guide](advanced-usage.md) and [API Reference](api-reference.md).