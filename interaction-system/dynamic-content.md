---
title: Dynamic Content
excerpt: Strategies for handling dynamic web content, JavaScript-rendered elements, and single-page applications.
meta:
  nav_order: 73
---

# Dynamic Content

Pixashot provides several strategies for handling dynamic content, ensuring accurate captures of JavaScript-rendered elements and single-page applications.

## Wait Strategies

Handle dynamically loaded content:

```json
{
  "url": "https://example.com",
  "custom_js": "
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

## Animation Handling

Wait for animations to complete:

```json
{
  "url": "https://example.com",
  "wait_for_animation": true,
  "delay_capture": 500
}
```

## Lazy Loading

Handle lazy-loaded images and content:

```json
{
  "url": "https://example.com",
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
    }
  ]
}
```

## SPA Navigation

Handle single-page application navigation:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "click",
      "selector": "#route-link"
    },
    {
      "action": "wait_for",
      "wait_for": {
        "type": "network_idle",
        "value": 2000
      }
    },
    {
      "action": "wait_for",
      "wait_for": {
        "type": "selector",
        "value": "#new-content"
      }
    }
  ]
}
```

## Best Practices

1. **Content Detection**
    - Monitor DOM changes
    - Watch network activity
    - Check rendering completion

2. **Resource Management**
    - Handle timeouts appropriately
    - Control network conditions
    - Monitor memory usage

3. **Error Prevention**
    - Implement fallback strategies
    - Handle state transitions
    - Validate content presence