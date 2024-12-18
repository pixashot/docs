---
title: Interaction System Overview
excerpt: Understanding Pixashot's page interaction capabilities for capturing dynamic web content.
meta:
  nav_order: 70
---

# Interaction System

Pixashot provides a powerful interaction system that allows you to manipulate web pages before capturing screenshots. This enables accurate capture of dynamic content, handling of popups, and complex multi-step interactions.

## Core Concepts

The interaction system is built on three key principles:

1. **Sequential Execution**: Actions are performed in order, with proper waiting between steps
2. **Error Recovery**: Robust error handling with retry capabilities
3. **Resource Efficiency**: Smart timeouts and resource management

## Basic Usage

Here's a simple example of using interactions:

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
    }
  ]
}
```

## Available Actions

- **Click**: Click on elements
- **Type**: Enter text into form fields
- **Hover**: Hover over elements
- **Scroll**: Scroll to specific positions
- **Wait**: Various wait strategies

## Error Handling

The interaction system includes built-in error handling:
- Automatic retries for failed actions
- Timeout management
- Detailed error reporting

## Integration

Interactions can be combined with other Pixashot features:
- Dark mode simulation
- Custom JavaScript execution
- Viewport configuration
- Network control

## Next Steps

- Learn about [Wait Strategies](wait-strategies.md)
- Explore [User Actions](user-actions.md)
- Understand [Dynamic Content](dynamic-content.md)