---
title: Advanced Screenshot Options
excerpt: Comprehensive guide to Pixashot's advanced capture features, including dark mode, geolocation, custom JavaScript, and complex interactions.
meta:
    nav_order: 32
---

# Advanced Screenshot Options

Learn how to use Pixashot's advanced features for complex screenshot scenarios, including dark mode capture, geolocation spoofing, and custom page interactions.

## Dark Mode Capture

Capture pages in dark mode with accurate color schemes:

```json
{
    "url": "https://example.com",
    "format": "png",
    "dark_mode": true,
    "wait_for_animation": true
}
```

### Dark Mode Features
- Automatic color scheme detection
- System preference emulation
- CSS media query handling
- Animation completion detection

### Best Practices
1. Wait for animations to complete
2. Allow time for theme switches
3. Verify color contrast
4. Test with different themes

## Geolocation Spoofing

Simulate specific geographic locations:

```json
{
    "url": "https://example.com",
    "format": "png",
    "geolocation": {
        "latitude": 37.7749,
        "longitude": -122.4194,
        "accuracy": 100
    }
}
```

### Location Testing
- Precise coordinate settings
- Accuracy configuration
- Permission handling
- Location-based content testing

## Custom JavaScript Injection

Execute custom JavaScript before capture:

```json
{
    "url": "https://example.com",
    "format": "png",
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
    "
}
```

### JavaScript Capabilities
- DOM manipulation
- Style modification
- Dynamic content handling
- Event simulation

## Complex Interaction Sequences

Define multi-step interaction sequences:

```json
{
    "url": "https://example.com",
    "format": "png",
    "interactions": [
        {
            "action": "click",
            "selector": "#accept-cookies"
        },
        {
            "action": "wait_for",
            "wait_for": {
                "type": "network_idle",
                "value": 2000
            }
        },
        {
            "action": "type",
            "selector": "#search",
            "text": "example query"
        },
        {
            "action": "wait_for",
            "wait_for": {
                "type": "selector",
                "value": "#results"
            }
        }
    ]
}
```

### Available Interactions
- Click events
- Text input
- Hover actions
- Scroll operations
- Custom delays
- Network waiting
- Element waiting

## Network Control

Advanced network handling and resource control:

```json
{
    "url": "https://example.com",
    "format": "png",
    "block_media": true,
    "ignore_https_errors": true,
    "custom_headers": {
        "User-Agent": "Custom User Agent",
        "Accept-Language": "en-US"
    },
    "wait_for_network": "mostly_idle"
}
```

### Network Options
- Media blocking
- HTTPS error handling
- Custom headers
- Network state monitoring
- Request timeout management

## Resource Optimization

Advanced resource management:

```json
{
    "url": "https://example.com",
    "format": "png",
    "block_media": true,
    "wait_for_network": "mostly_idle",
    "wait_for_timeout": 5000,
    "custom_js": "
        // Remove unnecessary resources
        document.querySelectorAll('video, iframe').forEach(el => el.remove());
        
        // Pause animations
        document.querySelectorAll('*').forEach(el => {
            el.style.animation = 'none';
            el.style.transition = 'none';
        });
    "
}
```

## Dynamic Content Handling

Advanced strategies for dynamic content:

```json
{
    "url": "https://example.com",
    "format": "png",
    "wait_for_network": "mostly_idle",
    "interactions": [
        {
            "action": "scroll",
            "x": 0,
            "y": 1000
        },
        {
            "action": "wait_for",
            "wait_for": {
                "type": "network_idle",
                "value": 2000
            }
        },
        {
            "action": "scroll",
            "x": 0,
            "y": 0
        }
    ],
    "wait_for_animation": true
}
```

## Advanced Error Handling

Comprehensive error handling strategies:

```json
{
    "url": "https://example.com",
    "format": "png",
    "ignore_https_errors": true,
    "wait_for_network": "mostly_idle",
    "wait_for_timeout": 10000,
    "interactions": [
        {
            "action": "wait_for",
            "wait_for": {
                "type": "selector",
                "value": "#content",
                "timeout": 5000
            }
        }
    ]
}
```

## Cookie and Popup Management

Advanced cookie and popup handling:

```json
{
    "url": "https://example.com",
    "format": "png",
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
        }
    ],
    "custom_js": "
        // Handle cookie preferences
        localStorage.setItem('cookie-preferences', 'accepted');
        
        // Remove popups
        document.querySelectorAll('[class*=popup], [class*=modal]')
            .forEach(el => el.remove());
    "
}
```

## Best Practices

1. **Resource Management**
    - Use appropriate wait strategies
    - Control media loading
    - Manage memory usage

2. **Error Handling**
    - Implement comprehensive retry logic
    - Handle timeouts appropriately
    - Log detailed error information

3. **Performance Optimization**
    - Minimize custom JavaScript
    - Use efficient selectors
    - Optimize wait conditions

4. **Security Considerations**
    - Validate custom JavaScript
    - Secure sensitive information
    - Control resource access

## Related Documentation

- [Output Format Options](output-formats.md)
- [Viewport Configuration](viewport-options.md)
- [Authentication](../security/authentication.md)
- [Error Handling](../troubleshooting/common-issues.md)