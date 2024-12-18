---
title: Output Format Configuration
excerpt: Detailed guide to Pixashot's output format options including PNG, JPEG, WebP, PDF, and HTML capture configurations.
meta:
  nav_order: 53
---

# Output Format Configuration

Pixashot supports multiple output formats, each with specific configuration options optimized for different use cases.

## PNG Format

PNG is the default format, offering lossless compression with alpha channel support.

```json
{
    "url": "https://example.com",
    "format": "png",
    "omit_background": true,
    "pixel_density": 2.0
}
```

### Transparency Options
- `omit_background`: Remove background for transparent PNGs
- Preserve alpha channels from source
- Maintain transparency in overlays
- Handle semi-transparent elements

### Resolution Control
- Adjustable pixel density for HiDPI
- Automatic DPI detection
- Custom scaling options
- Maximum dimensions: 16384x16384

### Best Practices
1. Use for:
    - Screenshots requiring perfect quality
    - Images with text
    - UI elements needing transparency
2. Consider:
    - File size implications
    - Memory usage for large captures
    - Browser compatibility

## JPEG Format

JPEG is ideal for photographs and complex images where some quality loss is acceptable.

```json
{
    "url": "https://example.com",
    "format": "jpeg",
    "image_quality": 80,
    "background_color": "#FFFFFF"
}
```

### Quality Settings
- `image_quality`: 0-100 (default: 90)
- Compression optimization
- Color subsampling options
- Size vs quality tradeoffs

### Color Optimization
- RGB color space
- Optional background color
- Color profile handling
- Gamma correction

### Size Optimization
1. Adjust quality based on needs
2. Balance quality vs file size
3. Consider content type
4. Monitor memory usage

## WebP Format

Modern format offering superior compression with good quality.

```json
{
    "url": "https://example.com",
    "format": "webp",
    "image_quality": 85,
    "omit_background": true
}
```

### Configuration Options
- Lossless or lossy compression
- Quality settings (0-100)
- Alpha channel support
- Animation preservation

### Performance Considerations
- Smaller file sizes than PNG/JPEG
- Faster transfer times
- Lower memory usage
- Better caching efficiency

## PDF Generation

Create professional PDF documents with extensive customization.

```json
{
    "url": "https://example.com",
    "format": "pdf",
    "pdf_format": "A4",
    "pdf_print_background": true,
    "pdf_scale": 1.0,
    "pdf_page_ranges": "1-5",
    "pdf_width": "210mm",
    "pdf_height": "297mm"
}
```

### Paper Sizes
- Standard formats:
    - A4 (210 × 297 mm)
    - Letter (8.5 × 11 inches)
    - Legal (8.5 × 14 inches)
- Custom dimensions:
    - Width and height in mm/inches
    - Custom page ranges
    - Margin control

### Page Configuration
- Background graphics
- Custom margins
- Page scaling
- Print adjustments

### Advanced Options
```json
{
    "url": "https://example.com",
    "format": "pdf",
    "pdf_format": "A4",
    "pdf_print_background": true,
    "wait_for_network": "idle",
    "custom_js": "document.body.classList.add('print-mode');"
}
```

## HTML Capture

Preserve complete HTML content with styles and resources.

```json
{
    "url": "https://example.com",
    "format": "html",
    "wait_for_network": "idle",
    "wait_for_animation": true
}
```

### Content Preservation
- Complete DOM structure
- Inline styles
- External resources
- Dynamic content

### Resource Handling
- CSS preservation
- Image embedding
- Font handling
- Script management

### Dynamic Content
```json
{
    "url": "https://example.com",
    "format": "html",
    "wait_for_network": "idle",
    "interactions": [
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

## Format Comparison

| Format | Transparency | Quality | File Size | Use Case |
|--------|--------------|----------|-----------|-----------|
| PNG | Yes | Lossless | Larger | UI, Text |
| JPEG | No | Configurable | Medium | Photos |
| WebP | Yes | Configurable | Smallest | Modern web |
| PDF | N/A | High | Varies | Documents |
| HTML | N/A | Original | Varies | Archives |

## Best Practices

1. **Format Selection**
    - PNG for quality-critical captures
    - JPEG for photographs
    - WebP for modern applications
    - PDF for documents
    - HTML for complete preservation

2. **Quality Optimization**
    - Match quality to use case
    - Consider bandwidth constraints
    - Monitor resource usage
    - Test with target devices

3. **Resource Management**
    - Set appropriate quality levels
    - Monitor memory usage
    - Implement caching
    - Handle cleanup properly

4. **Error Handling**
    - Validate format support
    - Check size limits
    - Monitor conversion errors
    - Implement fallbacks