---
title: Basic Screenshot Options
excerpt: Guide to essential screenshot capture features including full page captures, element selection, and basic wait strategies.
meta:
  nav_order: 51
---

# Basic Screenshot Options

Learn how to use Pixashot's core screenshot capabilities for common capture scenarios. This guide covers essential features suitable for most use cases.

## URL Capture

The simplest way to capture a screenshot is by providing a URL:

```json
{
    "url": "https://example.com",
    "format": "png"
}
```

### Required Parameters
- `url`: The webpage to capture (must be valid HTTP/HTTPS)
- `format`: Output format (`png` by default)

### Optional Basic Parameters
- `full_page`: Capture entire scrollable content
- `wait_for_network`: Wait for network activity to complete
- `block_media`: Skip loading images and media

## Full Page Screenshots

Capture the entire scrollable content of a webpage:

```json
{
    "url": "https://example.com",
    "format": "png",
    "full_page": true,
    "wait_for_network": "idle"
}
```

### Best Practices
1. Use `wait_for_network: "idle"` for dynamic content
2. Set appropriate viewport size
3. Consider memory usage for very long pages
4. Handle timeouts for large pages

## Element Capture

Target specific elements using CSS selectors:

```json
{
    "url": "https://example.com",
    "selector": "#main-content",
    "format": "png",
    "wait_for_selector": "#main-content"
}
```

### Element Selection Tips
- Use unique, stable selectors
- Wait for element visibility
- Handle dynamic content loading
- Consider element positioning

## HTML Content Capture

Render and capture HTML content directly:

```json
{
    "html_content": "<html><body><h1>Hello World</h1></body></html>",
    "format": "png"
}
```

### HTML Capture Features
- Direct HTML rendering
- Custom style injection
- Resource loading control
- Dynamic content support

## Basic Wait Strategies

Control when the screenshot is taken:

```json
{
    "url": "https://example.com",
    "format": "png",
    "wait_for_network": "idle",
    "wait_for_timeout": 5000,
    "wait_for_selector": "#content"
}
```

### Wait Options
- `wait_for_network`: Wait for network activity
- `wait_for_timeout`: Maximum wait time
- `wait_for_selector`: Wait for element
- `delay_capture`: Additional delay

## Error Handling

Common issues and solutions:

### Network Errors
```json
{
    "url": "https://example.com",
    "format": "png",
    "ignore_https_errors": true,
    "wait_for_timeout": 10000
}
```

### Missing Elements
```json
{
    "url": "https://example.com",
    "selector": "#content",
    "wait_for_selector": "#content",
    "wait_for_timeout": 5000
}
```

## Basic Templates

Use predefined templates for common scenarios:

### Mobile Template
```json
{
    "url": "https://example.com",
    "template": "mobile",
    "format": "png"
}
```

### Desktop Template
```json
{
    "url": "https://example.com",
    "template": "desktop_full",
    "format": "png"
}
```

## Performance Optimization

Basic performance tips:

1. **Resource Control**
   ```json
   {
       "url": "https://example.com",
       "format": "png",
       "block_media": true
   }
   ```

2. **Network Optimization**
   ```json
   {
       "url": "https://example.com",
       "format": "png",
       "wait_for_network": "mostly_idle"
   }
   ```

3. **Timeout Management**
   ```json
   {
       "url": "https://example.com",
       "format": "png",
       "wait_for_timeout": 5000
   }
   ```

## Common Use Cases

### Blog Post Capture
```json
{
    "url": "https://example.com/post",
    "format": "png",
    "full_page": true,
    "block_media": false,
    "wait_for_network": "idle"
}
```

### Product Image
```json
{
    "url": "https://example.com/product",
    "selector": "#product-image",
    "format": "png",
    "wait_for_network": "idle",
    "wait_for_selector": "#product-image"
}
```

## Next Steps

- Explore [Advanced Capture Features](advanced-capture.md)
- Learn about [Output Formats](output-formats.md)
- Configure [Viewport Options](viewport-options.md)

## Best Practices Summary

1. **Wait Strategies**
    - Use appropriate wait conditions
    - Set reasonable timeouts
    - Consider network activity

2. **Resource Usage**
    - Control media loading
    - Manage viewport size
    - Use appropriate formats

3. **Error Handling**
    - Implement retry logic
    - Handle timeouts gracefully
    - Validate input parameters