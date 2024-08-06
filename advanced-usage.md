# Advanced Usage

Pixashot offers advanced features for users who need more control over the capture process. These features allow for complex scenarios, custom manipulations, and handling of challenging web content.

## Custom JavaScript Injection

Inject custom JavaScript code to manipulate the page before capturing.

- **Powerful Customization**: Modify page content, styles, or behavior dynamically.
- **Pre-capture Automation**: Perform actions like clicking buttons or filling forms.
- **Usage**: Use the `custom_js` parameter to provide your JavaScript code.

Example:
```json
{
  "url": "https://example.com",
  "custom_js": "document.body.style.backgroundColor = 'lightblue'; document.querySelector('#popup').style.display = 'none';",
  "format": "png"
}
```

Best Practices:
- Ensure your JavaScript is error-free to avoid capture failures.
- Use this feature responsibly and in compliance with the target website's terms of service.

## CSS Manipulation

Modify the CSS of the page to change its appearance before capture.

- **Visual Adjustments**: Hide elements, change colors, or modify layouts.
- **Consistent Styling**: Ensure captured content looks exactly as needed.
- **Usage**: Inject CSS using the `custom_js` parameter.

Example:
```json
{
  "url": "https://example.com",
  "custom_js": "const style = document.createElement('style'); style.textContent = 'body { font-family: Arial, sans-serif; } .ad { display: none; }'; document.head.appendChild(style);",
  "format": "png"
}
```

Tips:
- Test your CSS changes thoroughly to ensure they don't break the page layout.
- Consider using `!important` for styles that need to override existing rules.

## Proxy Configuration

Route requests through a proxy server for accessing geo-restricted content or for anonymity.

- **Geo-restriction Bypass**: Access content as if from a different location.
- **Network Control**: Useful for testing or security purposes.
- **Usage**: Set `proxy_server`, `proxy_port`, and optionally `proxy_username` and `proxy_password`.

Example:
```json
{
  "url": "https://example.com",
  "proxy_server": "proxy.example.com",
  "proxy_port": 8080,
  "proxy_username": "user",
  "proxy_password": "pass",
  "format": "png"
}
```

Security Note:
- Always use secure, trusted proxies.
- Be cautious with proxy credentials in your requests.

## Cookie and Popup Handling

Pixashot provides built-in mechanisms to handle common web annoyances like cookie consent banners and popups.

- **Clean Captures**: Automatically remove or handle cookie consent popups.
- **Popup Blocking**: Prevent unwanted popups from interfering with captures.
- **Usage**: Enable or disable these features using `use_cookie_blocker` and `use_popup_blocker`.

Example:
```json
{
  "url": "https://example.com",
  "use_cookie_blocker": true,
  "use_popup_blocker": true,
  "format": "png"
}
```

Customization:
- If the built-in blockers are insufficient, use custom JavaScript to handle specific popups or banners.

## Handling Dynamic Content

Capture pages with dynamically loaded content accurately.

- **Wait Conditions**: Use various wait strategies to ensure content is loaded.
- **Custom Timing**: Set delays or wait for specific elements to appear.
- **Usage**: Utilize `wait_for_timeout`, `wait_for_selector`, or custom JavaScript for complex scenarios.

Example:
```json
{
  "url": "https://example.com",
  "wait_for_selector": "#dynamic-content",
  "wait_for_timeout": 5000,
  "custom_js": "window.scrollTo(0, document.body.scrollHeight);",
  "format": "png"
}
```

Best Practices:
- Combine multiple techniques for the most reliable captures of dynamic content.
- Use the `wait_for_network` option to ensure all network requests are complete.

Advanced Usage Tips:
1. **Debugging**: Use `custom_js` to log information to the console, which can be retrieved in error messages.
2. **Complex Interactions**: Combine custom JavaScript with wait conditions to perform multi-step interactions before capture.
3. **Resource Management**: Use `block_media` to prevent loading of images or videos if they're not needed in your capture.
4. **Error Handling**: Implement try-catch blocks in your custom JavaScript to gracefully handle potential errors.

By leveraging these advanced features, you can tackle complex web capture scenarios, ensure consistency across different websites, and create highly customized screenshots or PDFs tailored to your specific needs. Remember to test your advanced configurations thoroughly, especially when dealing with dynamic content or custom scripts.