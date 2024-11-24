---
title: Screenshot Capture Options
excerpt: Overview of Pixashot's screenshot capabilities, from basic captures to advanced customization features.
meta:
  nav_order: 30
---

# Screenshot Capture Options

Pixashot provides a comprehensive set of options for capturing web content, from simple screenshots to complex scenarios with custom interactions and outputs. As an open-source screenshot service, all features are freely available.

## Feature Overview

- [x] Full Page Screenshots
- [x] Element Selection
- [x] Custom Viewport Settings
- [x] PDF Export
- [x] Dark Mode Support
- [x] Geolocation Spoofing
- [x] Custom JavaScript Injection
- [x] Network Control
- [x] Multiple Output Formats

## Basic Capabilities

- **Full Page Capture**: Automatically detect and capture entire scrollable content
- **Element Selection**: Target specific elements using CSS selectors
- **Custom Viewports**: Set precise viewport dimensions and pixel density
- **Multiple Formats**: Support for PNG, JPEG, WebP, PDF, and HTML output

## Advanced Features

- **Dark Mode Support**: Accurate dark mode simulation and capture
- **Geolocation Spoofing**: Precise location simulation for testing
- **Custom JavaScript**: Pre-capture page manipulation
- **Network Control**: Advanced request handling and resource blocking

## Common Use Cases

### Visual Testing
```json
{
    "url": "https://example.com",
    "format": "png",
    "window_width": 1280,
    "window_height": 720,
    "pixel_density": 2
}
```

### PDF Generation
```json
{
    "url": "https://example.com",
    "format": "pdf",
    "pdf_format": "A4",
    "pdf_print_background": true,
    "full_page": true
}
```

### Mobile Capture
```json
{
    "url": "https://example.com",
    "format": "png",
    "window_width": 375,
    "window_height": 812,
    "pixel_density": 2,
    "user_agent_device": "mobile"
}
```

## Feature Categories

### Basic Capture Options
- URL-based capture
- HTML content rendering
- Element selection
- Simple wait strategies

### Advanced Configuration
- Custom JavaScript injection
- Cookie and popup handling
- Network simulation
- Resource blocking

### Output Formats
- PNG with transparency
- JPEG with quality control
- WebP for modern use
- PDF with customization
- HTML preservation

### Viewport Control
- Custom dimensions
- Device simulation
- Pixel density control
- Orientation options

## Getting Started

Choose your capture strategy based on your requirements:

1. **Simple Screenshots**: Use basic capture options for straightforward screenshots
2. **Advanced Scenarios**: Leverage advanced features for complex scenarios
3. **Output Optimization**: Configure format-specific options for optimal results
4. **Device Testing**: Use viewport options for cross-device testing

## Documentation Sections

- [Basic Capture Options](basic-capture.md): Learn about simple screenshot capture
- [Advanced Capture Features](advanced-capture.md): Explore complex capture scenarios
- [Output Format Configuration](output-formats.md): Detailed format settings
- [Viewport Configuration](viewport-options.md): Screen size and resolution options

## Best Practices

1. **Performance Optimization**
    - Use appropriate wait strategies
    - Enable caching when possible
    - Optimize viewport sizes

2. **Resource Management**
    - Control media loading
    - Manage memory usage
    - Handle timeouts properly

3. **Error Handling**
    - Implement retry logic
    - Monitor response codes
    - Log capture failures

4. **Quality Control**
    - Set appropriate pixel density
    - Configure format-specific options
    - Validate output requirements

## Memory Considerations

Since Pixashot uses a single browser context architecture, consider these factors:

1. **Resource Sharing**
    - Extensions are configured globally
    - Memory is shared across requests
    - Context settings apply to all captures

2. **Optimization Tips**
    - Use `block_media` when images aren't needed
    - Set appropriate viewport sizes
    - Implement proper cleanup
    - Monitor memory usage

3. **Concurrent Usage**
    - Configure worker count based on resources
    - Set appropriate request limits
    - Monitor system resources

For detailed information about specific features, explore the documentation sections above or refer to our [API Reference](../api-reference/index.md) guide.