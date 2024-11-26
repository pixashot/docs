---
title: Visual Regression Testing Guide
excerpt: Comprehensive guide for implementing visual regression testing with Pixashot, including CI/CD integration, test frameworks, and comparison strategies.
meta:
    nav_order: 132
---

# Visual Regression Testing

## Basic Implementation

### Component Testing
```python
class VisualTester:
    def __init__(self, base_url: str, threshold: float = 0.1):
        self.client = Client('your_api_key')
        self.base_url = base_url
        self.threshold = threshold
    
    async def test_component(self, component_path: str, states: list = ['default']):
        results = {}
        for state in states:
            screenshot = await self.client.capture({
                'url': f"{self.base_url}/{component_path}?state={state}",
                'format': 'png',
                'selector': '#component-root',
                'wait_for_network': 'idle',
                'custom_js': '''
                    // Remove dynamic content
                    document.querySelectorAll('[data-dynamic]').forEach(el => el.remove());
                    // Wait for animations
                    await new Promise(resolve => setTimeout(resolve, 1000));
                '''
            })
            results[state] = screenshot
        return results

```

## CI/CD Integration

### GitHub Actions Workflow
```yaml
name: Visual Tests
on: [pull_request]

jobs:
  visual-regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Capture Screenshots
        run: |
          pixashot compare \
            --base "origin/main" \
            --head "${{ github.sha }}" \
            --threshold 0.1 \
            --output reports/visual-diff
      
      - name: Upload Artifacts
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: visual-diff-report
          path: reports/visual-diff
```

## Comparison Strategies

```python
from PIL import Image
import numpy as np

async def compare_screenshots(baseline: bytes, current: bytes, threshold: float = 0.1):
    # Convert screenshots to numpy arrays
    baseline_img = Image.open(io.BytesIO(baseline))
    current_img = Image.open(io.BytesIO(current))
    
    baseline_array = np.array(baseline_img)
    current_array = np.array(current_img)
    
    # Calculate difference
    diff = np.mean(np.abs(baseline_array - current_array))
    return {
        'diff_percentage': diff,
        'exceeds_threshold': diff > threshold,
        'comparison_data': {
            'baseline_size': baseline_img.size,
            'current_size': current_img.size,
            'pixel_diff': diff
        }
    }
```

## Testing Patterns

### Responsive Testing
```python
VIEWPORTS = [
    {'width': 375, 'height': 667},  # Mobile
    {'width': 768, 'height': 1024}, # Tablet
    {'width': 1920, 'height': 1080} # Desktop
]

async def test_responsive_layout(url: str):
    client = Client('your_api_key')
    results = {}
    
    for viewport in VIEWPORTS:
        results[f"{viewport['width']}x{viewport['height']}"] = await client.capture({
            'url': url,
            'format': 'png',
            'window_width': viewport['width'],
            'window_height': viewport['height'],
            'pixel_density': 2.0,
            'wait_for_network': 'idle'
        })
    
    return results
```

### Theme Testing
```python
async def test_themes(url: str, themes: list = ['light', 'dark']):
    client = Client('your_api_key')
    results = {}
    
    for theme in themes:
        results[theme] = await client.capture({
            'url': f"{url}?theme={theme}",
            'format': 'png',
            'dark_mode': theme == 'dark',
            'wait_for_network': 'idle',
            'custom_js': f'document.documentElement.setAttribute("data-theme", "{theme}");'
        })
    
    return results
```

## Best Practices

1. **Test Configuration**
    - Set consistent viewport sizes
    - Use stable selectors
    - Remove dynamic content
    - Handle animations properly

2. **Performance**
    - Batch similar tests
    - Implement parallel processing
    - Cache when possible
    - Use appropriate timeouts

3. **Error Handling**
    - Implement retry logic
    - Log detailed errors
    - Save failure artifacts
    - Track flaky tests

For more details on specific features:
- [Capture Options](../capture-options/advanced-capture.md)
- [Viewport Configuration](../capture-options/viewport-options.md)
- [CI/CD Integration](../deployment/index.md)