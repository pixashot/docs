# Features

Pixashot offers a comprehensive set of features designed to capture web content with precision and flexibility. Whether you need full-page screenshots, specific element captures, or customized viewport sizes, Pixashot has you covered.

## Full Page Capture

Capture entire web pages, including content below the fold, with Pixashot's full page capture feature.

- **How it works**: Pixashot automatically scrolls through the entire page, capturing all content.
- **Usage**: Set `full_page: true` in your capture request.
- **Benefits**: Ideal for capturing long pages, ensuring no content is missed.

Example:
```json
{
  "url": "https://example.com",
  "full_page": true,
  "format": "png"
}
```

## Custom Viewport

Specify custom viewport dimensions to capture screenshots as they would appear on different devices.

- **Customizable**: Set both width and height of the viewport.
- **Device Simulation**: Easily simulate mobile, tablet, or desktop views.
- **Usage**: Use `window_width` and `window_height` parameters.

Example:
```json
{
  "url": "https://example.com",
  "window_width": 1280,
  "window_height": 720,
  "format": "png"
}
```

## Element Capture

Capture specific elements on a page using CSS selectors.

- **Precision**: Target exact elements for capture.
- **Flexibility**: Use any valid CSS selector.
- **Usage**: Specify the `selector` parameter in your request.

Example:
```json
{
  "url": "https://example.com",
  "selector": "#main-content",
  "format": "png"
}
```

## PDF Generation

Generate PDF documents from web pages with customizable options.

- **Full Control**: Adjust page size, orientation, and margins.
- **Background Graphics**: Option to include or exclude background graphics.
- **Usage**: Set `format: "pdf"` and use PDF-specific options.

Example:
```json
{
  "url": "https://example.com",
  "format": "pdf",
  "pdf_format": "A4",
  "pdf_print_background": true
}
```

## Multiple Formats (PNG, JPEG, WebP)

Capture screenshots in various image formats to suit your needs.

- **Supported Formats**: PNG, JPEG, and WebP.
- **Quality Control**: Adjust image quality for JPEG and WebP.
- **Usage**: Specify the desired format using the `format` parameter.

Example:
```json
{
  "url": "https://example.com",
  "format": "webp",
  "image_quality": 80
}
```

## Dark Mode Capture

Capture web pages in dark mode, simulating user preferences for dark themes.

- **Simulation**: Applies dark mode styles to the page before capture.
- **Compatibility**: Works with sites that support dark mode.
- **Usage**: Set `dark_mode: true` in your request.

Example:
```json
{
  "url": "https://example.com",
  "dark_mode": true,
  "format": "png"
}
```

## Geolocation Spoofing

Capture pages as they appear from different geographic locations by spoofing geolocation.

- **Precise Control**: Set exact latitude, longitude, and accuracy.
- **Testing**: Ideal for testing location-based content and features.
- **Usage**: Use the `geolocation` parameter with latitude, longitude, and accuracy.

Example:
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

## Custom Wait Conditions

Ensure the page is in the desired state before capturing by specifying custom wait conditions.

- **Network Idle**: Wait for network activity to settle.
- **Selector**: Wait for a specific element to appear.
- **Timeout**: Set a custom timeout duration.
- **Usage**: Use `wait_for_network`, `wait_for_selector`, or `wait_for_timeout` parameters.

Example:
```json
{
  "url": "https://example.com",
  "wait_for_selector": "#dynamic-content",
  "wait_for_network": "idle",
  "wait_for_timeout": 5000,
  "format": "png"
}
```

These features provide a powerful toolkit for capturing web content in various scenarios. Whether you're performing web testing, generating thumbnails, or archiving web pages, Pixashot's diverse feature set ensures you can capture exactly what you need, how you need it.