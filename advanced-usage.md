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

### Wait Strategies

Configure network and content waiting appropriately:

```json
{
  "url": "https://example.com",
  "wait_for_network": "idle",      // or "mostly_idle"
  "wait_for_timeout": 8000,        // milliseconds
  "wait_for_selector": "#content", // optional
  "format": "png"
}
```

Best practices:
- Use "mostly_idle" for dynamic content
- Set reasonable timeouts (default: 8000ms)
- Combine with selectors for specific elements

## Custom JavaScript Injection

Inject custom JavaScript code to manipulate the page before capturing.

Example:
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
  "format": "png"
}
```

Best Practices:
- Use async/await for timing-sensitive operations
- Handle errors gracefully
- Test thoroughly with different page states

## CSS Manipulation

Modify page appearance using CSS injection:

```json
{
  "url": "https://example.com",
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
      
      /* Ensure proper rendering */
      body {
        min-height: 100vh;
        overflow-x: hidden;
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

## Proxy Configuration

Configure proxy settings at the server level using environment variables:

```bash
# Server configuration
PROXY_SERVER=proxy.example.com
PROXY_PORT=8080
PROXY_USERNAME=user
PROXY_PASSWORD=pass
```

Security best practices:
- Use environment variables for credentials
- Implement proper access controls
- Monitor proxy usage and performance

## Cookie and Popup Management

Cookie and popup handling is configured server-wide through environment variables:

```bash
# Server configuration
USE_POPUP_BLOCKER=true
USE_COOKIE_BLOCKER=true
```

For custom handling:
```json
{
  "url": "https://example.com",
  "custom_js": "
    // Custom cookie handler
    window.localStorage.setItem('cookie-preferences', 'accepted');
    
    // Custom popup handler
    const config = { attributes: true, childList: true, subtree: true };
    const observer = new MutationObserver((mutations) => {
      document.querySelectorAll('[class*=popup], [class*=modal]')
        .forEach(el => el.remove());
    });
    observer.observe(document.body, config);
  ",
  "format": "png"
}
```

## Handling Dynamic Content

Comprehensive approach to capturing dynamic content:

```json
{
  "url": "https://example.com",
  "wait_for_network": "mostly_idle",
  "wait_for_timeout": 8000,
  "custom_js": "
    // Wait for dynamic content
    await new Promise(resolve => {
      let attempts = 0;
      const check = () => {
        attempts++;
        const content = document.querySelector('#dynamic-content');
        if (content || attempts > 20) {
          resolve();
        } else {
          setTimeout(check, 250);
        }
      };
      check();
    });

    // Ensure images are loaded
    await Promise.all(
      Array.from(document.images)
        .filter(img => !img.complete)
        .map(img => new Promise(resolve => {
          img.onload = img.onerror = resolve;
        }))
    );

    // Scroll to reveal lazy-loaded content
    await new Promise(resolve => {
      let position = 0;
      const scroll = () => {
        position += window.innerHeight;
        if (position >= document.body.scrollHeight) {
          window.scrollTo(0, 0);
          resolve();
        } else {
          window.scrollTo(0, position);
          setTimeout(scroll, 100);
        }
      };
      scroll();
    });
  ",
  "block_media": false,
  "format": "png"
}
```

## Performance Optimization

### Memory Management

```json
{
  "url": "https://example.com",
  "block_media": true,        // Block non-essential media
  "window_width": 1280,       // Reasonable viewport size
  "window_height": 720,
  "pixel_density": 1.0,       // Adjust based on needs
  "format": "png"
}
```

### Error Handling

```javascript
// Client-side retry logic
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
          wait_for_timeout: attempt * 5000, // Increase timeout with each retry
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

## Advanced Features

### Dark Mode Capture
```json
{
  "url": "https://example.com",
  "dark_mode": true,
  "wait_for_network": "idle",
  "format": "png"
}
```

### Geolocation Spoofing
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

Remember that Pixashot uses a single browser context, so:
- Extension settings apply to all requests
- Resources are shared efficiently
- Memory management is crucial
- Error handling should be robust

For more details on specific features, refer to the [API Reference](api-reference.md).