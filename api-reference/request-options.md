---
title: Request Options
excerpt: Complete reference for all available request parameters when using the Pixashot API, including viewport settings, format options, and interaction controls.
meta:
    nav_order: 72
---

# Request Options

Complete reference of all available parameters for the `/capture` endpoint.

## Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `url` | string | The webpage URL to capture (required if `html_content` not provided) |
| `html_content` | string | HTML to render and capture (required if `url` not provided) |

Either `url` or `html_content` must be provided, but not both.

## Basic Options

```json
{
  "url": "https://example.com",
  "format": "png",
  "window_width": 1280,
  "window_height": 720,
  "full_page": true,
  "wait_for_network": "idle"
}
```

## Viewport Options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `window_width` | integer | 1920 | Browser viewport width |
| `window_height` | integer | 1080 | Browser viewport height |
| `pixel_density` | float | 1.0 | Device scale factor (DPR) |
| `full_page` | boolean | false | Capture entire scrollable content |

## Format Options

| Parameter | Type | Values | Description |
|-----------|------|--------|-------------|
| `format` | string | png, jpeg, webp, pdf, html | Output format |
| `image_quality` | integer | 0-100 | Quality for JPEG/WebP |
| `omit_background` | boolean | false | Transparent background |

## Wait Strategies

| Parameter | Type | Description |
|-----------|------|-------------|
| `wait_for_network` | string | "idle" or "mostly_idle" |
| `wait_for_timeout` | integer | Maximum wait time (ms) |
| `wait_for_selector` | string | Wait for element to appear |
| `wait_for_animation` | boolean | Wait for animations to complete |

## PDF-Specific Options

```json
{
  "format": "pdf",
  "pdf_format": "A4",
  "pdf_print_background": true,
  "pdf_scale": 1.0,
  "pdf_page_ranges": "1-5",
  "pdf_width": "210mm",
  "pdf_height": "297mm"
}
```

## Advanced Options

| Parameter | Type | Description |
|-----------|------|-------------|
| `dark_mode` | boolean | Enable dark mode |
| `block_media` | boolean | Block images/media loading |
| `custom_js` | string | JavaScript to execute |
| `custom_headers` | object | Additional HTTP headers |
| `ignore_https_errors` | boolean | Ignore SSL errors |

## Page Interactions

```json
{
  "interactions": [
    {
      "action": "click",
      "selector": "#button"
    },
    {
      "action": "type",
      "selector": "#input",
      "text": "Hello"
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

## Geolocation Options

```json
{
  "geolocation": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "accuracy": 100
  }
}
```

## Device Simulation

```json
{
  "user_agent_device": "mobile",
  "user_agent_platform": "ios",
  "user_agent_browser": "safari",
  "window_width": 375,
  "window_height": 812,
  "pixel_density": 2.0
}
```

See [Viewport Options](../capture-options/viewport-options.md) for detailed device presets.