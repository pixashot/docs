---
title: Viewport Configuration
excerpt: Comprehensive guide to viewport configuration including device simulation, resolution handling, and responsive design testing.
meta:
    nav_order: 34
---

# Viewport Configuration

Configure viewport settings to capture screenshots at specific dimensions and resolutions.

## Standard Viewport Sizes

Common viewport configurations for different devices and scenarios:

```json
{
    "url": "https://example.com",
    "window_width": 1920,
    "window_height": 1080,
    "pixel_density": 1.0
}
```

### Desktop Presets
- 1920×1080 (Full HD)
- 1366×768 (Laptop)
- 1440×900 (Desktop)
- 2560×1440 (QHD)

### Tablet Presets
- 1024×768 (iPad)
- 834×1112 (iPad Pro)
- 1280×800 (Android Tablet)

### Mobile Presets
- 375×812 (iPhone)
- 390×844 (iPhone 13)
- 360×740 (Android)

## Mobile Device Simulation

Accurately simulate mobile devices with correct settings:

```json
{
    "url": "https://example.com",
    "window_width": 375,
    "window_height": 812,
    "pixel_density": 2.0,
    "user_agent_device": "mobile",
    "user_agent_platform": "ios"
}
```

### Device Configurations
- User agent customization
- Screen orientation
- Touch event simulation
- Platform-specific behavior

### Common Mobile Presets
```json
{
    "url": "https://example.com",
    "template": "iphone_13",  // Predefined template
    "format": "png"
}
```

## HiDPI/Retina Support

Configure high-resolution captures for retina displays:

```json
{
    "url": "https://example.com",
    "window_width": 1920,
    "window_height": 1080,
    "pixel_density": 2.0,  // 2x resolution
    "format": "png"
}
```

### Resolution Configuration
- `pixel_density`: Device scale factor (1.0-3.0)
- Physical vs logical pixels
- Screen density handling
- Memory considerations

### Best Practices
1. Match target device density
2. Consider memory limitations
3. Balance quality vs performance
4. Test across devices

## Responsive Design Testing

Test responsive layouts across breakpoints:

```json
{
    "url": "https://example.com",
    "window_width": 375,
    "window_height": 812,
    "pixel_density": 2.0,
    "user_agent_device": "mobile",
    "custom_js": "document.documentElement.style.setProperty('--vh', '${window_height}px');"
}
```

### Breakpoint Testing
- Common breakpoints:
    - 320px (Mobile S)
    - 375px (Mobile M)
    - 425px (Mobile L)
    - 768px (Tablet)
    - 1024px (Laptop)
    - 1440px (Desktop)

### Device Rotation
```json
{
    "url": "https://example.com",
    "window_width": 812,    // Swapped dimensions
    "window_height": 375,   // for landscape
    "pixel_density": 2.0,
    "user_agent_device": "mobile"
}
```

## Size Limitations

Understanding and working with size constraints:

### Maximum Dimensions
- Max viewport width: 16384px
- Max viewport height: 16384px
- Max pixel density: 3.0
- Memory limits based on device

### Handling Large Captures
```json
{
    "url": "https://example.com",
    "window_width": 1920,
    "window_height": 1080,
    "full_page": true,
    "block_media": true,    // Optimize memory
    "format": "jpeg",       // More efficient for large sizes
    "image_quality": 85     // Balance quality vs size
}
```

## Performance Considerations

Optimize viewport configuration for performance:

### Memory Usage
- Larger viewports use more memory
- Higher pixel density increases memory usage
- Full-page captures require additional resources
- Media blocking can reduce memory usage

### Best Practices
1. **Resource Management**
   ```json
   {
       "url": "https://example.com",
       "window_width": 1920,
       "window_height": 1080,
       "block_media": true,
       "wait_for_network": "mostly_idle"
   }
   ```

2. **Large Screen Capture**
   ```json
   {
       "url": "https://example.com",
       "window_width": 2560,
       "window_height": 1440,
       "pixel_density": 1.0,    // Reduced for large screens
       "format": "jpeg",
       "image_quality": 85
   }
   ```

3. **Mobile Optimization**
   ```json
   {
       "url": "https://example.com",
       "window_width": 375,
       "window_height": 812,
       "pixel_density": 2.0,
       "block_media": true,
       "wait_for_network": "mostly_idle"
   }
   ```

## Resolution Best Practices

Guidelines for optimal resolution configuration:

1. **Desktop Captures**
    - Use 1x pixel density for standard screens
    - Use 2x for HiDPI content
    - Match target display resolution
    - Consider memory constraints

2. **Mobile Captures**
    - Always use 2x or 3x for mobile
    - Match device pixel ratios
    - Consider orientation changes
    - Test with actual devices

3. **Memory Management**
    - Reduce pixel density for large viewports
    - Block unnecessary resources
    - Use efficient output formats
    - Monitor system resources

4. **Quality Control**
    - Verify text clarity
    - Check image sharpness
    - Test across devices
    - Validate output sizes

Remember that Pixashot uses a single browser context, so resource usage should be carefully monitored when capturing at high resolutions or with large viewports.