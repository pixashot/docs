---
title: Common Use Cases
excerpt: Overview of practical applications and implementation patterns for Pixashot across different industries and scenarios.
meta:
    nav_order: 70
---

# Common Use Cases

Pixashot serves diverse use cases across different industries, from e-commerce to web archiving. This guide explores common implementation patterns and real-world applications.

## Key Applications

### E-commerce
- Automated product image capture
- Dynamic preview generation
- Product comparison screenshots
- Social media catalog generation
  [Learn more](e-commerce.md)

### Visual Regression Testing
- Automated UI testing
- Layout verification
- Cross-browser testing
- Design system validation
  [Learn more](testing.md)

### Web Archiving
- Content preservation
- Legal compliance documentation
- Knowledge base snapshots
- Historical record keeping
  [Learn more](archiving.md)

## Industry Solutions

### Marketing & Social Media
```json
{
    "url": "https://example.com/article",
    "format": "png",
    "window_width": 1200,
    "window_height": 630,
    "pixel_density": 2.0,
    "wait_for_network": "idle"
}
```
- Social media preview images
- Marketing material generation
- Campaign asset creation
- Blog post thumbnails

### SaaS & Development
```json
{
    "url": "https://app.example.com",
    "format": "png",
    "full_page": true,
    "wait_for_selector": "#app-content",
    "block_media": true
}
```
- UI regression testing
- Documentation screenshots
- Bug report captures
- Feature previews

### Legal & Compliance
```json
{
    "url": "https://example.com/terms",
    "format": "pdf",
    "pdf_format": "A4",
    "full_page": true,
    "wait_for_network": "idle"
}
```
- Terms of service archives
- Compliance documentation
- Legal evidence capture
- Policy documentation

## Integration Examples

### Automated Systems
```python
from pixashot import Client

client = Client('your_api_key')

async def capture_product(product_url, product_id):
    screenshot = await client.capture({
        'url': product_url,
        'format': 'png',
        'selector': '#product-image',
        'wait_for_network': 'idle'
    })
    await save_to_cdn(screenshot, product_id)
```

### CI/CD Pipeline
```yaml
name: Visual Tests
on: [pull_request]

jobs:
  visual-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Compare Screenshots
        run: |
          pixashot compare \
            --base "main" \
            --head "current" \
            --threshold 0.1
```

## Key Features by Use Case

| Feature              | E-commerce | Testing | Archiving |
|---------------------|------------|----------|-----------|
| Full Page Capture   | ✓          | ✓        | ✓         |
| Element Selection   | ✓          | ✓        | -         |
| Custom JavaScript   | ✓          | ✓        | -         |
| PDF Export         | -          | -        | ✓         |
| Network Control    | ✓          | ✓        | ✓         |
| Mobile Viewport    | ✓          | ✓        | -         |

For detailed implementation guides, explore the specific use case documentation:
- [E-commerce Guide](e-commerce.md)
- [Testing Guide](testing.md)
- [Archiving Guide](archiving.md)