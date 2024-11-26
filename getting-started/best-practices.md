---
title: Pixashot Best Practices
excerpt: Recommended practices for optimizing performance, reliability, and security when using Pixashot.
meta:
    nav_order: 24
---

# Pixashot Best Practices

This guide covers recommended practices for using Pixashot effectively in production environments, focusing on performance, reliability, and security.

## Resource Management

### Memory Optimization

```bash
# Recommended memory settings by usage
docker run -p 8080:8080 \
  --memory=4g \           # Production minimum
  --memory-reservation=2g # Soft limit
  gpriday/pixashot:latest
```

Best practices for memory management:
- Start with 2GB minimum memory allocation
- Monitor memory usage through health endpoints
- Configure `MAX_REQUESTS` to recycle workers periodically
- Use `WORKERS` count appropriate for available memory

Memory guidelines by traffic:
```
Light traffic    (< 10 req/min):    2GB
Medium traffic   (10-50 req/min):   4GB
Heavy traffic    (50+ req/min):     8GB+
```

### Worker Configuration

Optimal worker settings:
```bash
# Light usage
export WORKERS=2
export MAX_REQUESTS=500

# Medium usage
export WORKERS=4
export MAX_REQUESTS=1000

# Heavy usage
export WORKERS=8
export MAX_REQUESTS=2000
```

Worker count formula:
```
WORKERS = MIN(CPU_CORES * 2, AVAILABLE_MEMORY_GB)
```

## Performance Optimization

### Capture Settings

Optimize capture speed with these settings:

```json
{
  "url": "https://example.com",
  "format": "png",
  "wait_for_network": "mostly_idle",
  "block_media": true,
  "window_width": 1280,
  "window_height": 720,
  "pixel_density": 1.0
}
```

For faster captures:
- Use `wait_for_network: "mostly_idle"` instead of `"idle"`
- Enable `block_media` when images aren't needed
- Use appropriate viewport sizes
- Set reasonable `wait_for_timeout` values
- Use `pixel_density: 1.0` unless higher resolution needed

### Caching Strategy

Enable and configure caching for repeated requests:

```bash
# Production caching configuration
export CACHE_MAX_SIZE=1000        # Cache size based on memory
export RATE_LIMIT_ENABLED=true    # Enable rate limiting
export RATE_LIMIT_CAPTURE="5 per second"
```

Caching considerations:
- Enable for frequently accessed pages
- Set cache size based on available memory
- Monitor cache hit rates
- Implement client-side caching when appropriate

## Request Handling

### Error Handling

Implement robust error handling:

```javascript
async function captureWithRetry(url, maxAttempts = 3) {
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            const response = await fetch('/capture', {
                method: 'POST',
                headers: {
                    'Authorization': `Bearer ${AUTH_TOKEN}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({
                    url,
                    wait_for_network: 'mostly_idle',
                    wait_for_timeout: attempt * 5000
                })
            });

            if (!response.ok) throw new Error(`HTTP ${response.status}`);
            return response;

        } catch (error) {
            if (attempt === maxAttempts) throw error;
            await new Promise(r => setTimeout(r, Math.pow(2, attempt) * 1000));
        }
    }
}
```

Error handling best practices:
- Implement exponential backoff
- Set appropriate timeouts
- Handle specific error types
- Log errors with context
- Monitor error rates

### Rate Limiting

Configure rate limits based on infrastructure:

```bash
# Recommended rate limiting configuration
export RATE_LIMIT_ENABLED=true
export RATE_LIMIT_CAPTURE="5 per second"    # Adjust based on capacity
export RATE_LIMIT_SIGNED="10 per second"    # Higher limit for trusted clients
```

Client-side rate limit handling:

```javascript
class RateLimitHandler {
    constructor(maxRetries = 3) {
        this.maxRetries = maxRetries;
        this.retryDelays = new Map();
    }

    async handleRequest(requestFn) {
        let attempts = 0;
        while (attempts < this.maxRetries) {
            try {
                return await requestFn();
            } catch (error) {
                if (error.status === 429) {
                    const retryAfter = parseInt(error.headers.get('Retry-After') || '5');
                    await this.delay(retryAfter * 1000);
                    attempts++;
                } else {
                    throw error;
                }
            }
        }
        throw new Error('Rate limit retry attempts exceeded');
    }

    async delay(ms) {
        await new Promise(resolve => setTimeout(resolve, ms));
    }
}
```

## Security Best Practices

### Authentication

Secure token management:

```bash
# Generate secure token
export AUTH_TOKEN=$(openssl rand -hex 32)

# Store token in secret management service
gcloud secrets create pixashot-auth-token --data-file=- <<< "$AUTH_TOKEN"
```

Security recommendations:
- Rotate tokens regularly
- Use separate tokens for different environments
- Store tokens in secret management services
- Monitor token usage

### Network Security

Configure network security:

```bash
# Production security settings
export HTTPS_ONLY=true
export IGNORE_HTTPS_ERRORS=false
export PROXY_SERVER=your-proxy-server
```

## Monitoring and Logging

### Health Checks

Implement comprehensive health monitoring:

```bash
# Health check script
#!/bin/bash

check_health() {
    response=$(curl -s http://localhost:8080/health)
    memory_usage=$(echo $response | jq .checks.memory_usage_mb)
    cpu_percent=$(echo $response | jq .checks.cpu_percent)

    if [ $memory_usage -gt 90 ]; then
        echo "High memory usage: $memory_usage%"
        exit 1
    fi

    if [ $cpu_percent -gt 80 ]; then
        echo "High CPU usage: $cpu_percent%"
        exit 1
    fi
}
```

### Logging

Configure appropriate logging:

```bash
# Production logging configuration
export LOG_LEVEL=warning      # Reduce noise in production
export LOG_FORMAT=json        # Structured logging for analysis
```

## Request Optimization

### Wait Strategies

Choose appropriate wait strategies:

```json
{
  "url": "https://example.com",
  "wait_for_network": "mostly_idle",
  "interactions": [
    {
      "action": "wait_for",
      "wait_for": {
        "type": "selector",
        "value": "#content"
      }
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

Wait strategy recommendations:
- Use `mostly_idle` for dynamic content
- Set reasonable timeouts
- Combine multiple wait strategies when needed
- Monitor timeout frequencies

### Resource Control

Control resource usage:

```json
{
  "url": "https://example.com",
  "block_media": true,
  "format": "jpeg",
  "image_quality": 80,
  "pixel_density": 1.0
}
```

Resource optimization tips:
- Block unnecessary resources
- Use appropriate image formats
- Optimize quality settings
- Set suitable viewport sizes

## Production Deployment

### Docker Deployment

```bash
# Production Docker configuration
docker run -d \
  --name pixashot \
  --restart=unless-stopped \
  --memory=4g \
  --cpus=2 \
  -e AUTH_TOKEN=$AUTH_TOKEN \
  -e WORKERS=4 \
  -e MAX_REQUESTS=1000 \
  -e RATE_LIMIT_ENABLED=true \
  -e RATE_LIMIT_CAPTURE="5 per second" \
  -e CACHE_MAX_SIZE=1000 \
  -e LOG_LEVEL=warning \
  -p 8080:8080 \
  gpriday/pixashot:latest
```

### Cloud Run Deployment

```bash
# Production Cloud Run configuration
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --memory 4Gi \
  --cpu 2 \
  --min-instances 1 \
  --max-instances 10 \
  --concurrency 80 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=$AUTH_TOKEN,WORKERS=4"
```

## Maintenance

Regular maintenance tasks:
1. Monitor resource usage
2. Review error logs
3. Update configurations
4. Rotate authentication tokens
5. Update Pixashot version

## Testing and Validation

Regularly test:
- Screenshot quality
- Error handling
- Rate limiting
- Resource usage
- Authentication
- Network resilience

## Next Steps

After implementing these best practices:
1. Set up [Monitoring](../deployment/monitoring.md)
2. Configure [Alerts](../deployment/monitoring.md#alerts)
3. Review [Security](../security/index.md)
4. Plan [Scaling](../deployment/scaling.md)

For more advanced configurations, see the [Advanced Usage Guide](../features/advanced-features.md).