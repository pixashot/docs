---
title: Resource Management
excerpt: Comprehensive guide to how Pixashot manages memory, CPU, and browser resources for optimal performance and reliability.
meta:
    nav_order: 33
---

# Resource Management

Pixashot's resource management system ensures efficient operation while preventing memory leaks and resource exhaustion. This guide explains how resources are allocated, monitored, and optimized.

## Memory Management

### Worker Process Management

Pixashot uses a worker-based system for request handling:

```python
# Environment configuration
WORKERS = int(os.getenv('WORKERS', 4))           # Number of worker processes
MAX_REQUESTS = int(os.getenv('MAX_REQUESTS', 1000))  # Requests before worker restart
KEEP_ALIVE = int(os.getenv('KEEP_ALIVE', 300))   # Keep-alive timeout
```

Each worker process:
- Handles a subset of incoming requests
- Is recycled after `MAX_REQUESTS`
- Maintains its own memory space
- Shares the browser context

### Memory Monitoring

Memory usage is tracked through the health endpoint:

```python
@app.route('/health')
async def health_check():
    process = psutil.Process(os.getpid())
    memory_usage = process.memory_info().rss / 1024 / 1024  # MB
    
    return {
        'status': 'healthy',
        'checks': {
            'memory_usage_mb': round(memory_usage, 2),
            'cpu_percent': round(process.cpu_percent(), 2)
        }
    }
```

### Page Lifecycle

Pages are created and destroyed for each request:

```python
async def capture_screenshot(self, output_path, options):
    try:
        # Create new page
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

## Browser Resource Management

### Context Initialization

The browser context is initialized with optimized settings:

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

### Extension Resource Management

Extensions are loaded once and shared:

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

Network connections are managed efficiently:

```python
async def _resilient_navigation(self, page: Page, url: str, timeout: int):
    try:
        await page.goto(
            str(url),
            wait_until='domcontentloaded',  # Less strict wait condition
            timeout=timeout
        )
    except Exception as nav_error:
        logger.warning(f"Navigation timeout: {str(nav_error)}. Continuing...")
```

### Proxy Configuration

Proxy settings are managed at the context level:

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

Optional media blocking for improved performance:

```python
async def configure_resource_blocking(self, page: Page, options):
    if options.block_media:
        await page.route("**/*.{png,jpg,jpeg,gif,webp,svg,mp4,webm,ogg,mp3,wav}",
            lambda route: route.abort())
```

### Cache Management

Response caching for improved performance:

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

## Resource Limits

### Memory Limits

Memory limits are enforced through Docker configuration:

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

Timeouts prevent resource exhaustion:

```python
async def take_screenshot(self, page: Page, options: dict) -> bytes:
    screenshot_options = {
        'timeout': self.SCREENSHOT_TIMEOUT_MS,  # Default: 10000ms
        'type': options.get('format', 'png'),
        'quality': options.get('quality')
    }
    
    try:
        return await self._take_screenshot_with_retry(page, screenshot_options)
    except TimeoutError:
        return await self._take_fallback_screenshot(page, screenshot_options)
```

## Monitoring and Recovery

### Health Checks

Regular health monitoring:

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

Automatic cleanup on errors:

```python
async def cleanup_resources(self):
    """Clean up resources after capture."""
    try:
        # Clean temporary files
        if os.path.exists(self.temp_path):
            os.remove(self.temp_path)
            
        # Reset page state
        await self.page.evaluate("window.stop()")
        await self.page.reload()
        
    except Exception as e:
        logger.error(f"Cleanup error: {str(e)}")
```

## Best Practices

### Memory Optimization

1. **Worker Configuration**
   ```bash
   # For 4-core system
   WORKERS=4
   MAX_REQUESTS=1000
   KEEP_ALIVE=300
   ```

2. **Resource Limits**
   ```bash
   # Production settings
   --memory=2g
   --cpu=1
   ```

3. **Cache Settings**
   ```bash
   # Enable caching for repeated requests
   CACHE_MAX_SIZE=1000
   ```

### Monitoring Tips

1. **Memory Usage**
    - Monitor `/health` endpoint
    - Set up alerts for high memory usage
    - Track memory trends

2. **Performance Metrics**
    - Monitor request latency
    - Track error rates
    - Monitor CPU usage

3. **Resource Alerts**
    - Set up alerts for resource exhaustion
    - Monitor worker recycling
    - Track cleanup failures

## Troubleshooting

### Common Issues

1. **Memory Leaks**
    - Check MAX_REQUESTS setting
    - Monitor worker recycling
    - Verify cleanup procedures

2. **High CPU Usage**
    - Check worker count
    - Monitor request patterns
    - Review browser arguments

3. **Resource Exhaustion**
    - Verify memory limits
    - Check timeout settings
    - Monitor network resources

## Next Steps

- Learn about [Performance Optimization](../deployment/scaling.md)
- Understand [Monitoring](../deployment/cloud-run.md)
- Explore [Advanced Features](../api-reference/request-options.md)

Effective resource management is crucial for:
- Reliable operation
- Consistent performance
- Cost efficiency
- Scaling capabilities

Understanding and properly configuring resource management ensures optimal Pixashot operation in production environments.