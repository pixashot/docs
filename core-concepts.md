# Core Concepts

Understanding the core concepts of Pixashot will help you make the most of its capabilities. This section covers how Pixashot works under the hood, provides an overview of the API, and explains the request flow.

## How Pixashot Works

Pixashot uses advanced browser automation technology to capture high-quality screenshots of web pages. Here's a detailed explanation of its architecture and operation:

### Single-Context Architecture

Pixashot uses a single browser context architecture for optimal performance and resource utilization:

1. **Browser Initialization**:
   - One Playwright browser instance is launched during startup
   - Browser extensions (popup blocker, cookie consent blocker) are configured via environment variables
   - A single browser context is created and maintained throughout the service lifetime
   - User agent settings are managed globally for consistency

2. **Worker Processes**:
   - Multiple worker processes handle incoming requests (configurable via `WORKERS`)
   - Each worker shares the same browser context
   - Workers are recycled after handling a configured number of requests (`MAX_REQUESTS`)
   - Memory usage is monitored and managed automatically

### Screenshot Process

When a request is received:

1. **Request Validation**:
   - Parameters are validated using Pydantic models
   - Template application (if specified)
   - URL security validation
   - Authentication verification

2. **Page Creation**:
   - New page created within shared browser context
   - Custom headers and user agent configured
   - Geolocation settings applied (if specified)

3. **Page Configuration**:
   - Viewport size and pixel density set
   - Network conditions configured
   - Dark mode enabled (if requested)
   - Resource blocking applied (if specified)

4. **Content Loading**:
   - URL loaded or HTML content rendered
   - Network state monitored
   - Animation completion tracked
   - Dynamic content handling

5. **Interaction Sequence**:
   - Structured interaction steps executed
   - Network idle checks performed
   - Element visibility verified
   - Custom JavaScript executed

6. **Page Customization**:
   - Dark mode application
   - Custom JavaScript injection
   - Style modifications
   - Print optimization (for PDF)

7. **Screenshot Capture**:
   - Format-specific preparation
   - Quality settings applied
   - Image optimization
   - PDF configuration (if applicable)

8. **Cleanup**:
   - Page resources released
   - Memory management
   - Context maintenance
   - Error logging

## API Overview

Pixashot provides a RESTful API with a single endpoint and comprehensive customization options.

### Main Endpoint

```
POST /capture
```

### Key Parameter Categories

#### Basic Options
- `url` or `html_content`: Source content
- `template`: Optional template name
- `format`: Output format (png, jpeg, webp, pdf, html)

#### Viewport and Screenshot
- `window_width`: Viewport width
- `window_height`: Viewport height
- `full_page`: Full page capture
- `selector`: Element-specific capture

#### User Agent Configuration
- `use_random_user_agent`: Enable random UA
- `user_agent_device`: Device type (desktop/mobile)
- `user_agent_platform`: OS platform
- `user_agent_browser`: Browser type

#### Interaction Options
- `interactions`: Structured interaction steps
- `wait_for_animation`: Animation completion
- `wait_for_network`: Network state handling

#### Image Settings
- `image_quality`: Quality level
- `pixel_density`: Display scaling
- `dark_mode`: Dark mode rendering
- `omit_background`: Transparency

#### PDF-specific Options
- `pdf_format`: Page format
- `pdf_print_background`: Background printing
- `pdf_scale`: Content scaling
- `pdf_page_ranges`: Page selection

### Example Request

```json
{
  "url": "https://example.com",
  "template": "mobile",
  "format": "png",
  "wait_for_animation": true,
  "interactions": [
    {
      "action": "wait_for",
      "wait_for": {
        "type": "network_idle",
        "value": 2000
      }
    },
    {
      "action": "click",
      "selector": "#accept-cookies"
    }
  ],
  "dark_mode": true,
  "pixel_density": 2.0
}
```

### Environment Configuration

Server-wide settings configured through environment variables:

- `USE_POPUP_BLOCKER`: Popup blocking
- `USE_COOKIE_BLOCKER`: Cookie consent blocking
- `RATE_LIMIT_ENABLED`: Rate limiting
- `CACHE_MAX_SIZE`: Response caching
- `WORKERS`: Process count
- `MAX_REQUESTS`: Worker lifecycle

## Request Flow

Here's the complete flow of a Pixashot request:

1. **Request Processing**:
   - Parameter validation
   - Template application
   - Security checks
   - Authentication

2. **Page Setup**:
   - Context configuration
   - Device simulation
   - Network settings
   - Resource management

3. **Content Loading**:
   ```mermaid
   graph TD
     A[Request Received] --> B[Validate Parameters]
     B --> C[Apply Template]
     C --> D[Configure Page]
     D --> E[Load Content]
     E --> F[Execute Interactions]
     F --> G[Wait for Completion]
     G --> H[Capture Content]
     H --> I[Process Result]
     I --> J[Send Response]
   ```

4. **Resource Management**:
   - Memory optimization
   - Context reuse
   - Worker recycling
   - Error recovery

5. **Response Handling**:
   - Format processing
   - Quality optimization
   - Header configuration
   - Error reporting

### Performance Considerations

The single-context architecture provides several benefits:

1. **Resource Efficiency**:
   - Shared browser context
   - Memory optimization
   - Fast page creation
   - Efficient resource usage

2. **Consistent Behavior**:
   - Standardized extensions
   - Predictable rendering
   - Reliable performance
   - Consistent user agent handling

3. **Scalability**:
   - Process-based concurrency
   - Resource sharing
   - Memory management
   - Load distribution

4. **Limitations**:
   - Server-wide extensions
   - Shared context constraints
   - Memory management requirements
   - Process cycling needs

## Best Practices

1. **Resource Optimization**:
   - Use templates for common scenarios
   - Enable media blocking when appropriate
   - Configure viewport sizes carefully
   - Optimize interaction sequences

2. **Error Handling**:
   - Implement retry strategies
   - Monitor timeouts
   - Check health endpoints
   - Log errors appropriately

3. **Performance Tuning**:
   - Balance worker count
   - Configure request limits
   - Enable caching where appropriate
   - Use appropriate wait strategies

4. **Security**:
   - Use HTTPS
   - Implement rate limiting
   - Secure authentication tokens
   - Validate input URLs

Understanding these core concepts helps you optimize your use of Pixashot and troubleshoot any issues that arise. For more detailed information about specific features, see our [Advanced Usage](advanced-usage.md) and [API Reference](api-reference.md) documentation.