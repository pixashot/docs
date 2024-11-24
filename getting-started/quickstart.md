---
title: Quick Start with Pixashot
excerpt: Take your first screenshots and learn the basics of using Pixashot with practical examples.
meta:
    nav_order: 30
---

# Quick Start Guide

This guide will walk you through taking your first screenshots with Pixashot, demonstrating the most common use cases and features.

## Your First Screenshot

Let's start with a basic screenshot:

```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png"
  }'
```

This will return a PNG image of the viewport. The response will be the binary image data which you can save to a file.

## Common Capture Scenarios

### Full Page Screenshot

Capture an entire webpage, including scrolled content:

```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true,
    "wait_for_network": "idle"
  }'
```

### Capture Specific Element

Target a specific element using CSS selectors:

```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "selector": "#main-content",
    "wait_for_selector": "#main-content"
  }'
```

### Generate PDF

Convert a webpage to PDF:

```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "pdf",
    "pdf_format": "A4",
    "pdf_print_background": true,
    "full_page": true
  }'
```

### Mobile Device Simulation

Capture a mobile viewport:

```bash
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer your_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "window_width": 375,
    "window_height": 812,
    "pixel_density": 2.0,
    "user_agent_device": "mobile",
    "user_agent_platform": "ios"
  }'
```

## Using Client Libraries

### Python Example

```python
import requests

def capture_screenshot(url, output_path):
    response = requests.post(
        'http://localhost:8080/capture',
        headers={
            'Authorization': 'Bearer your_token',
            'Content-Type': 'application/json'
        },
        json={
            'url': url,
            'format': 'png',
            'full_page': True,
            'wait_for_network': 'idle'
        }
    )
    
    if response.status_code == 200:
        with open(output_path, 'wb') as f:
            f.write(response.content)
        print(f"Screenshot saved to {output_path}")
    else:
        print(f"Error: {response.status_code}")
        print(response.json())

# Usage
capture_screenshot('https://example.com', 'screenshot.png')
```

### Node.js Example

```javascript
const axios = require('axios');
const fs = require('fs').promises;

async function captureScreenshot(url, outputPath) {
    try {
        const response = await axios.post('http://localhost:8080/capture', {
            url,
            format: 'png',
            full_page: true,
            wait_for_network: 'idle'
        }, {
            headers: {
                'Authorization': 'Bearer your_token',
                'Content-Type': 'application/json'
            },
            responseType: 'arraybuffer'
        });

        await fs.writeFile(outputPath, response.data);
        console.log(`Screenshot saved to ${outputPath}`);
    } catch (error) {
        console.error('Screenshot failed:', error.message);
        if (error.response) {
            console.error('Status:', error.response.status);
        }
    }
}

// Usage
captureScreenshot('https://example.com', 'screenshot.png');
```

## Handling Dynamic Content

For pages with dynamic content, use wait strategies:

```json
{
  "url": "https://example.com",
  "format": "png",
  "wait_for_network": "idle",
  "wait_for_animation": true,
  "interactions": [
    {
      "action": "wait_for",
      "wait_for": {
        "type": "selector",
        "value": "#dynamic-content"
      }
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

## Handling Common Scenarios

### Cookie Consent and Popups

Pixashot includes built-in popup and cookie consent blockers. They're enabled by default if configured in your environment:

```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest
```

### Custom JavaScript Injection

Modify page content before capture:

```json
{
  "url": "https://example.com",
  "format": "png",
  "custom_js": "document.querySelectorAll('.advertisement').forEach(el => el.remove());",
  "wait_for_network": "idle"
}
```

### Dark Mode Capture

Capture pages in dark mode:

```json
{
  "url": "https://example.com",
  "format": "png",
  "dark_mode": true,
  "wait_for_animation": true
}
```

## Error Handling

Common error scenarios and how to handle them:

### Network Timeout
```json
{
  "url": "https://example.com",
  "format": "png",
  "wait_for_timeout": 10000,
  "wait_for_network": "mostly_idle"
}
```

### Missing Elements
```json
{
  "url": "https://example.com",
  "format": "png",
  "selector": "#content",
  "wait_for_selector": "#content",
  "wait_for_timeout": 5000
}
```

## Best Practices for Getting Started

1. **Start Simple**
    - Begin with basic screenshots
    - Add features incrementally
    - Test thoroughly before adding complexity

2. **Test Different Formats**
    - Try PNG for quality
    - Use JPEG for smaller files
    - Test PDF for documents

3. **Monitor Performance**
    - Watch response times
    - Check resource usage
    - Adjust settings as needed

4. **Handle Errors**
    - Implement retry logic
    - Use appropriate timeouts
    - Log failures for debugging

## Next Steps

Once you're comfortable with basic captures:

1. Review [Configuration](configuration.md) for detailed settings
2. Check [Best Practices](best-practices.md) for optimization
3. Explore [Advanced Features](../features/advanced-features.md)
4. Setup proper [Error Handling](../api-reference/error-handling.md)

For more detailed API information, visit our [API Reference](../api-reference/index.md).