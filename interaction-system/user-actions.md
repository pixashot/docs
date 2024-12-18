---
title: User Actions
excerpt: Guide to simulating user interactions like clicks, typing, and scrolling with Pixashot.
meta:
  nav_order: 72
---

# User Actions

Pixashot can simulate various user actions to interact with web pages before capturing screenshots. This enables capturing content that requires user interaction to display.

## Click Actions

Click specific elements:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "click",
      "selector": "#submit-button"
    }
  ]
}
```

## Text Input

Enter text into form fields:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "type",
      "selector": "#search-input",
      "text": "Example search"
    }
  ]
}
```

## Hover Interactions

Trigger hover states:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "hover",
      "selector": "#dropdown-menu"
    }
  ]
}
```

## Scroll Operations

Control page scrolling:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "scroll",
      "x": 0,
      "y": 500
    }
  ]
}
```

## Complex Sequences

Combine multiple actions:

```json
{
  "url": "https://example.com",
  "interactions": [
    {
      "action": "click",
      "selector": "#accept-cookies"
    },
    {
      "action": "type",
      "selector": "#username",
      "text": "testuser"
    },
    {
      "action": "click",
      "selector": "#submit"
    }
  ]
}
```

## Best Practices

1. **Selector Reliability**
    - Use unique, stable selectors
    - Test selectors across page states
    - Handle dynamic content changes

2. **Action Timing**
    - Add appropriate waits between actions
    - Consider animation completion
    - Handle loading states

3. **Error Handling**
    - Plan for missing elements
    - Include timeout fallbacks
    - Monitor action success