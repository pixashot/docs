---
title: Resource Management
excerpt: Comprehensive guide to how Pixashot manages memory, CPU, and browser resources for optimal performance and reliability.
meta:
  nav_order: 33
---

# Resource Management

Pixashot's resource management system is designed to ensure efficient operation while preventing memory leaks, CPU exhaustion, and other resource-related issues. This guide explains how resources are allocated, monitored, and optimized within the Pixashot architecture.

## Memory Management

### Worker Process Management

Pixashot utilizes a worker-based system for handling concurrent requests. The number of worker processes is configurable via the `WORKERS` environment variable:

```python
# Default worker configuration in src/config/__init__.py
WORKERS = int(os.getenv('WORKERS', 4))  # Number of worker processes
MAX_REQUESTS = int(os.getenv('MAX_REQUESTS', 1000)) # Requests before worker restart
KEEP_ALIVE = int(os.getenv('KEEP_ALIVE', 300))  # Keep-alive timeout in seconds
```

Each worker process:

- Handles a subset of incoming requests.
- Is recycled after processing `MAX_REQUESTS` requests to prevent memory leaks.
- Maintains its own memory space, separate from other workers.
- Shares the same browser context, managed by the `ContextManager`.

### Memory Monitoring

Memory usage is tracked and exposed through the `/health` endpoint. The `psutil` library is used to gather process-specific memory information:

```python
@app.route('/health')
async def health_check():
    process = psutil.Process(os.getpid())
    memory_usage = process.memory_info().rss / 1024 / 1024  # in MB
    
    return {
        'status': 'healthy',
        'checks': {
            'memory_usage_mb': round(memory_usage, 2),
            'cpu_percent': round(process.cpu_percent(), 2)
        }
    }
```

### Page Lifecycle Management

To minimize memory usage per request, each screenshot request creates a new page within the shared browser context and ensures its closure after processing:

```python
async def capture_screenshot(self, output_path, options):
    try:
        page = await self.context.new_page()
        try:
            await self._configure_page(page, options)
            # Capture screenshot
            await self.screenshot_controller.take_screenshot(page, options)
        finally:
            # Ensure page cleanup
            await page.close()
    except Exception as e:
        raise ScreenshotServiceException(str(e))
```

This ensures that resources used by the page are released promptly after each request.

## Browser Resource Management

### Context Initialization

The `ContextManager` initializes the browser context with optimized settings to reduce resource usage:

```python
async def initialize(self, playwright):
    browser_args = [
        '--disable-features=site-per-process',
        '--no-sandbox',
        '--disable-setuid-sandbox',
        '--disable-dev-shm-usage',
    ]

    self.browser = await playwright.chromium.launch(args=browser_args)
    context_options = {
        'viewport': {'width': 1920, 'height': 1080},
        'device_scale_factor': 1.0
    }
    
    self.context = await self.browser.new_context(**context_options)
```

These settings disable unnecessary features and optimize memory usage.

### Extension Resource Management

Extensions (popup blocker, cookie consent handler) are loaded once and shared across all requests, reducing the startup overhead:

```python
def _get_extension_args(self):
    extensions = []
    if self.use_popup_blocker:
        extensions.append(os.path.join(self.extension_dir, 'popup-off'))
    if self.use_cookie_blocker:
        extensions.append(os.path.join(self.extension_dir, 'dont-care-cookies'))

    if not extensions:
        return []

    return [
        f'--disable-extensions-except={",".join(extensions)}',
        *[f'--load-extension={ext}' for ext in extensions]
    ]
```

## Network Resource Management

### Connection Pooling

Playwright's underlying browser context maintains a pool of network connections, which are reused across requests. This reduces the overhead of establishing new connections for each request.

### Proxy Configuration

Proxy settings are managed at the context level, allowing for efficient routing of network traffic:

```python
def _get_proxy_config(self):
    if config.PROXY_SERVER and config.PROXY_PORT:
        proxy_config = {
            'server': f'{config.PROXY_SERVER}:{config.PROXY_PORT}'
        }
        
        if config.PROXY_USERNAME and config.PROXY_PASSWORD:
            proxy_config.update({
                'username': config.PROXY_USERNAME,
                'password': config.PROXY_PASSWORD
            })
        
        return proxy_config
    return None
```

## Resource Optimization

### Media Blocking

Pixashot allows blocking of media resources (images, videos, etc.) to reduce bandwidth and processing overhead:

```python
async def configure_resource_blocking(self, page: Page, options):
    if options.block_media:
        await page.route("**/*.{png,jpg,jpeg,gif,webp,svg,mp4,webm,ogg,mp3,wav}", lambda route: route.abort())
```

### Cache Management

A configurable response cache (disabled by default) can be enabled to reduce redundant processing for repeated requests:

```python
class CacheManager:
    def __init__(self, max_size=None):
        self.max_size = max_size
        self.cache = lru_cache(maxsize=max_size)(lambda k: k)() if max_size else None

    def get(self, key):
        return self.cache.get(key) if self.cache else None

    def set(self, key, value):
        if self.cache:
            self.cache[key] = value
```

This uses an LRU (Least Recently Used) cache to store responses based on the request URL.

## Resource Limits

### Memory Limits

Memory limits are enforced at the container level through Docker or container orchestration platforms like Kubernetes or Cloud Run:

```yaml
# Docker deployment example
docker run -p 8080:8080 \
-e AUTH_TOKEN=your_secret_token \
-e WORKERS=4 \
-e MAX_REQUESTS=1000 \
--memory=2g \
gpriday/pixashot:latest
```

### Request Timeouts

Timeouts are used to prevent long-running requests from consuming resources indefinitely:

```python
async def take_screenshot(self, page: Page, options: dict) -> bytes:
    screenshot_options = {
        'timeout': self.SCREENSHOT_TIMEOUT_MS,  # Default: 10000ms (configurable)
        'type': options.get('format', 'png'),
        'quality': options.get('quality')
    }
    # ... rest of the screenshot logic ...
```

## Monitoring and Recovery

### Health Checks

Regular health monitoring is exposed through the `/health` and `/health/ready` endpoints, providing insights into memory usage, CPU utilization, and overall system status.

```python
@app.route('/health/ready')
async def readiness_check():
    try:
        process = psutil.Process(os.getpid())
        memory_info = process.memory_info()
        
        return {
            'status': 'ready',
            'memory': {
                'rss_mb': memory_info.rss / 1024 / 1024,
                'vms_mb': memory_info.vms / 1024 / 1024,
                'percent': process.memory_percent()
            },
            'cpu_percent': process.cpu_percent()
        }
    except Exception as e:
        return {'status': 'not_ready', 'error': str(e)}, 503
```

### Resource Recovery

Automatic cleanup routines are in place to release resources after each request, even in case of errors:

```python
async def capture_screenshot(self, output_path, options):
    page = None  # Initialize page to None
    try:
        page = await self.context.new_page()
        # ... rest of the capture logic ...
    except Exception as e:
        logger.error(f"Screenshot capture error: {str(e)}")
        raise ScreenshotServiceException(str(e))
    finally:
        if page:
          await page.close()
        # Ensure cleanup of temporary files
        if os.path.exists(output_path):
            os.remove(output_path)
```

## Best Practices

### Memory Optimization

1. **Worker Configuration**:

    ```bash
    # Example for a system with 4 CPU cores and 8GB RAM:
    WORKERS=4  # Matches the number of CPU cores
    MAX_REQUESTS=1000 # Recycle workers after 1000 requests
    KEEP_ALIVE=300 # Adjust keep-alive timeout based on expected load
    ```

2. **Resource Limits**:

    ```bash
    # Example Docker Compose configuration
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 2G
    ```

3. **Cache Settings**:

    ```bash
    # Enable caching for repeated requests (if applicable)
    CACHE_MAX_SIZE=1000 # Cache up to 1000 responses
    ```

### Monitoring Tips

1. **Memory Usage**:
    - Monitor the `/health` endpoint for memory usage metrics.
    - Set up alerts for high memory usage (e.g., >80% utilization).
    - Track memory usage trends over time to identify potential leaks.

2. **Performance Metrics**:
    - Monitor request latency to identify performance bottlenecks.
    - Track error rates to identify issues with specific requests or configurations.
    - Monitor CPU usage to ensure optimal worker configuration.

3. **Resource Alerts**:
    - Set up alerts for resource exhaustion (e.g., memory or CPU limits reached).
    - Monitor worker recycling frequency to identify potential issues.
    - Track cleanup failures to detect resource leaks.

## Troubleshooting

### Common Issues

1. **Memory Leaks**:
    - Verify that `MAX_REQUESTS` is set appropriately to recycle workers.
    - Monitor memory usage over time to identify gradual increases.
    - Review custom JavaScript code for potential memory leaks.

2. **High CPU Usage**:
    - Check the number of `WORKERS` and adjust based on CPU cores.
    - Monitor CPU usage and identify any spikes or sustained high usage.
    - Review browser arguments for potential optimizations.

3. **Resource Exhaustion**:
    - Verify memory and CPU limits in your deployment configuration.
    - Check timeout settings to prevent long-running requests.
    - Monitor network resources to ensure they are not a bottleneck.

## Next Steps

- Learn about [Performance Optimization](../deployment/scaling.md) for scaling your deployment and improving efficiency.
- Understand [Monitoring](../deployment/cloud-run.md#monitoring-setup) for setting up monitoring and alerts.
- Explore [Advanced Features](../api-reference/request-options.md) to leverage the full capabilities of Pixashot.

Effective resource management is crucial for:

- **Reliable operation**: Preventing crashes and ensuring consistent service availability.
- **Consistent performance**: Maintaining optimal response times under various loads.
- **Cost efficiency**: Optimizing resource utilization to minimize operational costs.
- **Scaling capabilities**: Enabling the system to handle increased traffic efficiently.

By understanding and properly configuring Pixashot's resource management features, you can ensure optimal operation in production environments, efficiently handle varying workloads, and provide a reliable screenshot service to your users.