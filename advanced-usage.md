---
excerpt: "Advanced techniques for using Pixashot, including custom JavaScript injection, CSS manipulation, proxy configuration, and handling dynamic content."
published_at: true
---

# Advanced Usage

Pixashot offers advanced features for users who need more control over the capture process. These features allow for complex scenarios, custom manipulations, and handling of challenging web content.

## Request Optimization

Understanding Pixashot's single-context architecture is crucial for advanced usage:

- Single browser context shared across requests
- Extensions configured via environment variables
- Consistent behavior across captures
- Efficient resource utilization

### Advanced Wait Strategies

Configure network and content waiting using interaction steps:

```json
{
  "url": "https://example.com",
  "wait_for_network": "idle",
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
        "type": "network_mostly_idle",
        "value": 2000
      }
    }
  ],
  "wait_for_animation": true,
  "format": "png"
}
```

Best practices:
- Use "mostly_idle" for dynamic content
- Combine multiple wait strategies for complex pages
- Set reasonable timeouts (default: 8000ms)
- Use `wait_for_animation` for pages with CSS transitions

## User Agent Management

Control browser identification with user agent settings:

```json
{
  "url": "https://example.com",
  "use_random_user_agent": true,
  "user_agent_device": "mobile",
  "user_agent_platform": "ios",
  "user_agent_browser": "safari",
  "window_width": 375,
  "window_height": 812,
  "pixel_density": 2.0,
  "format": "png"
}
```

Tips:
- Match viewport settings to device type
- Use appropriate pixel density for device simulation
- Consider platform-specific features

## Custom JavaScript Injection

Inject custom JavaScript code to manipulate the page before capturing:

```json
{
  "url": "https://example.com",
  "custom_js": "
    // Remove unwanted elements
    document.querySelectorAll('.ad, .popup').forEach(el => el.remove());
    
    // Modify styles
    document.body.style.backgroundColor = 'white';
    
    // Wait for dynamic content
    await new Promise(resolve => {
      const observer = new MutationObserver(() => {
        if (document.querySelector('#dynamic-content')) {
          observer.disconnect();
          resolve();
        }
      });
      observer.observe(document.body, { childList: true, subtree: true });
    });
  ",
  "wait_for_animation": true,
  "format": "png"
}
```

Best Practices:
- Use async/await for timing-sensitive operations
- Handle errors gracefully
- Test thoroughly with different page states
- Combine with interaction steps for complex scenarios

## CSS Manipulation

Modify page appearance using CSS injection:

```json
{
  "url": "https://example.com",
  "dark_mode": true,
  "custom_js": "
    const style = document.createElement('style');
    style.textContent = `
      /* Hide unwanted elements */
      .ad, .popup, .cookie-notice { display: none !important; }
      
      /* Improve capture quality */
      * { 
        animation: none !important;
        transition: none !important;
        scroll-behavior: auto !important;
      }
      
      /* Dark mode enhancements */
      @media (prefers-color-scheme: dark) {
        body {
          background: #121212;
          color: #ffffff;
        }
      }
      
      /* Print optimizations */
      @media print {
        body {
          -webkit-print-color-adjust: exact;
          print-color-adjust: exact;
        }
      }
    `;
    document.head.appendChild(style);
  ",
  "format": "png"
}
```

Tips:
- Use `!important` when necessary
- Consider print-specific styles for PDF captures
- Handle responsive layouts appropriately
- Combine with dark mode setting for consistent results

## Complex Interaction Sequences

Handle complex user interactions before capture:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "click",
      "selector": "#cookie-accept"
    },
    {
      "action": "wait_for",
      "wait_for": {
        "type": "network_idle",
        "value": 1000
      }
    },
    {
      "action": "type",
      "selector": "#search",
      "text": "example search"
    },
    {
      "action": "click",
      "selector": "#search-button"
    },
    {
      "action": "wait_for",
      "wait_for": {
        "type": "selector",
        "value": ".search-results"
      }
    },
    {
      "action": "scroll",
      "x": 0,
      "y": 500
    },
    {
      "action": "hover",
      "selector": ".dropdown-menu"
    }
  ],
  "wait_for_animation": true,
  "format": "png"
}
```

## Advanced Dynamic Content Handling

Comprehensive approach to capturing dynamic content:

```json
{
  "url": "https://example.com",
  "wait_for_network": "mostly_idle",
  "interactions": [
    {
      "action": "wait_for",
      "wait_for": {
        "type": "selector",
        "value": "#dynamic-content"
      }
    }
  ],
  "custom_js": "
    // Ensure images are loaded
    await Promise.all(
      Array.from(document.images)
        .filter(img => !img.complete)
        .map(img => new Promise(resolve => {
          img.onload = img.onerror = resolve;
        }))
    );

    // Handle infinite scroll
    await new Promise(resolve => {
      let lastHeight = 0;
      const checkHeight = () => {
        const currentHeight = document.body.scrollHeight;
        if (currentHeight === lastHeight) {
          resolve();
        } else {
          lastHeight = currentHeight;
          window.scrollTo(0, currentHeight);
          setTimeout(checkHeight, 250);
        }
      };
      checkHeight();
    });

    // Return to top
    window.scrollTo(0, 0);
  ",
  "wait_for_animation": true,
  "block_media": false,
  "format": "png"
}
```

## PDF Generation

Advanced PDF configuration:

```json
{
  "url": "https://example.com",
  "format": "pdf",
  "pdf_format": "A4",
  "pdf_print_background": true,
  "pdf_scale": 0.8,
  "pdf_page_ranges": "1-5",
  "full_page": true,
  "wait_for_network": "idle",
  "custom_js": "
    // Add print-specific styles
    const style = document.createElement('style');
    style.textContent = `
      @page {
        margin: 2cm;
      }
      @media print {
        .no-print { display: none !important; }
        pre, code {
          white-space: pre-wrap;
          word-break: break-word;
        }
      }
    `;
    document.head.appendChild(style);
  "
}
```

## Error Handling and Retries

```javascript
// Advanced retry logic with interaction handling
async function captureWithRetry(url, maxAttempts = 3) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const response = await fetch('/capture', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${AUTH_TOKEN}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          url,
          wait_for_network: 'mostly_idle',
          wait_for_timeout: attempt * 5000,
          interactions: [
            {
              action: 'wait_for',
              wait_for: {
                type: 'network_idle',
                value: attempt * 2000
              }
            }
          ],
          wait_for_animation: true,
          format: 'png'
        })
      });

      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response;

    } catch (error) {
      if (attempt === maxAttempts) throw error;
      await new Promise(r => setTimeout(r, Math.pow(2, attempt) * 1000));
    }
  }
}
```

## Templates for Common Scenarios

Use templates to standardize capture configurations:

```json
{
  "url": "https://example.com",
  "template": "mobile",  // Uses predefined mobile template
  "custom_js": "...",   // Additional customizations
  "format": "png"
}
```

Remember that Pixashot uses a single browser context, so:
- Extension settings apply to all requests
- Resources are shared efficiently
- Memory management is crucial
- Error handling should be robust

For more details on specific features, refer to the [API Reference](api-reference.md).