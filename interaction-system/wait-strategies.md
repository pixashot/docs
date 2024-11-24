---
title: Wait Strategies
excerpt: Comprehensive guide to Pixashot's wait strategies for handling dynamic content and network activity.
meta:
nav_order: 41
---

# Wait Strategies

Pixashot offers multiple wait strategies to ensure your screenshots capture the right content at the right time.

## Network Wait Strategies

### Network Idle
Wait for network to be completely idle:

```json
{
  "url": "https://example.com",
  "wait_for_network": "idle",
  "format": "png"
}
```

### Mostly Idle
More lenient network waiting, suitable for sites with continuous background activity:

```json
{
  "url": "https://example.com",
  "wait_for_network": "mostly_idle",
  "format": "png"
}
```

## Element Wait Strategies

Wait for specific elements to appear:

```json
{
  "url": "https://example.com",
  "wait_for_selector": "#content",
  "format": "png"
}
```

## Timeout Controls

Manage wait timeouts explicitly:

```json
{
  "url": "https://example.com",
  "wait_for_timeout": 5000,  // Maximum 5 seconds
  "format": "png"
}
```

## Combined Strategies

Use multiple wait strategies for complex scenarios:

```json
{
  "url": "https://example.com",
  "format": "png",
  "wait_for_network": "idle",
  "wait_for_selector": "#dynamic-content",
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

## Best Practices

1. **Network Waiting**
    - Use "mostly_idle" for sites with analytics
    - Set reasonable timeouts
    - Consider connection speed

2. **Element Waiting**
    - Use specific selectors
    - Include timeout fallbacks
    - Handle dynamic content

3. **Performance**
    - Balance wait times with speed
    - Use appropriate strategies
    - Monitor timeout frequency