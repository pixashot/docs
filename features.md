---
excerpt: "Complete overview of Pixashot's capabilities, including screenshot capture options, page interactions, format support, and advanced features."
published_at: true
---

# Features

Pixashot is a powerful web screenshot service that offers a comprehensive set of features for capturing web content with precision and flexibility. This document outlines all available features and their usage.

## Core Screenshot Capabilities

### Full Page Capture
- Automatically captures entire scrollable page content
- Smart scroll detection for dynamic content
- Configurable maximum page height (16384px by default)
- Handles lazy-loaded content

```json
{
  "url": "https://example.com",
  "full_page": true,
  "format": "png",
  "wait_for_network": "idle"
}
```

### Element Capture
- Target specific page elements using CSS selectors
- Automatic element visibility detection
- Handles dynamically loaded elements
- Supports multiple selector formats

```json
{
  "url": "https://example.com",
  "selector": "#main-content",
  "format": "png",
  "wait_for_selector": "#main-content"
}
```

### Viewport Control
- Customizable viewport dimensions
- High-DPI support with adjustable pixel density
- Mobile device viewport simulation
- Automatic viewport adjustment for full-page captures

```json
{
  "url": "https://example.com",
  "window_width": 1280,
  "window_height": 720,
  "pixel_density": 2.0,
  "format": "png"
}
```

## Output Formats

### Image Formats
- **PNG**: Lossless quality with transparency support
- **JPEG**: Adjustable quality compression
- **WebP**: Modern format with superior compression
- Configurable image quality settings
- Optional background transparency

```json
{
  "url": "https://example.com",
  "format": "jpeg",
  "image_quality": 80,
  "omit_background": false
}
```

### PDF Generation
- Professional PDF document creation
- Multiple paper size options (A4, Letter, Legal)
- Configurable page margins and scaling
- Optional background graphics
- Custom page ranges support

```json
{
  "url": "https://example.com",
  "format": "pdf",
  "pdf_format": "A4",
  "pdf_print_background": true,
  "pdf_scale": 1.0,
  "pdf_page_ranges": "1-5"
}
```

### HTML Capture
- Complete HTML document capture
- Optional resource inclusion
- Dark mode preservation
- Dynamic content handling

```json
{
  "url": "https://example.com",
  "format": "html",
  "wait_for_network": "idle"
}
```

## Page Interaction Features

### Wait Conditions
- Network activity monitoring
- Element presence detection
- Custom timeout configuration
- Animation completion detection

```json
{
  "url": "https://example.com",
  "wait_for_network": "idle",
  "wait_for_selector": "#dynamic-content",
  "wait_for_timeout": 5000,
  "wait_for_animation": true
}
```

### Custom JavaScript
- Pre-capture JavaScript execution
- DOM manipulation support
- Dynamic content handling
- Custom styling injection

```json
{
  "url": "https://example.com",
  "custom_js": "document.body.style.backgroundColor = 'white';",
  "format": "png"
}
```

### Interaction Sequences
- Click simulation
- Text input
- Hover actions
- Custom scroll positions
- Complex multi-step interactions

```json
{
  "url": "https://example.com",
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
  ],
  "format": "png"
}
```

## Advanced Features

### Dark Mode Support
- Dark mode simulation
- System preference emulation
- CSS scheme handling
- Consistent dark mode capture

```json
{
  "url": "https://example.com",
  "dark_mode": true,
  "format": "png"
}
```

### Geolocation Spoofing
- Precise location simulation
- Configurable accuracy
- Full geolocation API support
- Location-based testing

```json
{
  "url": "https://example.com",
  "geolocation": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "accuracy": 100
  },
  "format": "png"
}
```

### Resource Control
- Optional media blocking
- Popup blocking (configurable)
- Cookie consent handling
- Network request management

```json
{
  "url": "https://example.com",
  "block_media": true,
  "format": "png"
}
```

### User Agent Control
- Random user agent generation
- Device type specification
- Platform customization
- Browser version control

```json
{
  "url": "https://example.com",
  "use_random_user_agent": true,
  "user_agent_device": "desktop",
  "user_agent_platform": "windows",
  "user_agent_browser": "chrome",
  "format": "png"
}
```

## Performance Features

### Request Optimization
- Single browser context architecture
- Efficient resource sharing
- Memory usage optimization
- Request queuing and management

### Caching Support
- Optional response caching
- Configurable cache size
- Cache invalidation controls
- Performance optimization

### Rate Limiting
- Configurable rate limits
- Separate limits for different endpoints
- Rate limit headers
- Graceful handling of limits

## Security Features

### Authentication
- Token-based authentication
- Signed URL support
- Flexible authentication options
- Secure token management

### Network Security
- HTTPS error handling
- Proxy support with authentication
- Custom header support
- Request validation

```json
{
  "url": "https://example.com",
  "proxy_server": "proxy.example.com",
  "proxy_port": 8080,
  "proxy_username": "user",
  "proxy_password": "pass",
  "format": "png"
}
```

## Health Monitoring

### Health Checks
- Liveness probe endpoint
- Readiness probe endpoint
- Detailed health status
- Resource usage monitoring

These features make Pixashot a versatile tool for various use cases, from automated testing to content archival. For detailed implementation examples, refer to our [Example Use Cases](example-use-cases.md) documentation.