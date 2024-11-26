---
title: Performance and Scaling
excerpt: Guidelines for scaling Pixashot deployments, including CPU and memory optimization, worker configuration, and performance tuning.
meta:
    nav_order: 154
---

# Performance and Scaling

This guide covers scaling considerations and performance optimization for Pixashot deployments.

## Resource Requirements

### Base Requirements Per Instance
- CPU: 1 core minimum
- RAM: 2GB minimum
- Storage: 1GB base + temporary storage
- Network: Reliable outbound connection

### Scaling Formula
```
Workers = MIN(CPU_CORES * 2, AVAILABLE_MEMORY_GB)
Concurrent_Requests = Workers * 10
```

## Memory Scaling

### Memory Per Request Type
```
Standard Screenshot:   ~50MB
Full Page:            ~100MB
PDF Generation:       ~150MB
Complex Interactions: ~200MB
```

### Instance Sizing
```bash
# Light load (<10 req/min)
--memory=2g --workers=2

# Medium load (10-50 req/min)
--memory=4g --workers=4

# Heavy load (50+ req/min)
--memory=8g --workers=8
```

## Worker Configuration

### Environment Variables
```bash
# Light usage
export WORKERS=2
export MAX_REQUESTS=500
export KEEP_ALIVE=300

# Medium usage
export WORKERS=4
export MAX_REQUESTS=1000
export KEEP_ALIVE=300

# Heavy usage
export WORKERS=8
export MAX_REQUESTS=2000
export KEEP_ALIVE=300
```

## Performance Optimization

### Browser Context
```json
{
    "block_media": true,            // Reduce memory usage
    "wait_for_network": "mostly_idle",  // Faster than "idle"
    "pixel_density": 1.0,          // Lower for large viewports
    "format": "jpeg",              // More efficient than PNG
    "image_quality": 85            // Balance quality vs size
}
```

### Caching
```bash
# Enable response caching
export CACHE_MAX_SIZE=1000        # Adjust based on memory
export RATE_LIMIT_ENABLED=true    # Prevent overload
export RATE_LIMIT_CAPTURE="5 per second"
```

## Cloud Run Auto-scaling

### Configuration
```bash
# Optimal scaling settings
gcloud run deploy pixashot \
  --min-instances=1 \
  --max-instances=10 \
  --concurrency=80 \
  --cpu=2 \
  --memory=4Gi \
  --timeout=300
```

### Load Balancing
- Set appropriate `min-instances` for steady traffic
- Configure `max-instances` based on peak load
- Use `concurrency` to control requests per instance
- Monitor cold start performance

## Monitoring Metrics

Key metrics to track:
- Memory usage per request
- CPU utilization
- Response times
- Error rates
- Cold start frequency
- Cache hit rates

### Health Endpoints
```bash
# Monitor these endpoints
/health          # Overall health
/health/live     # Liveness check
/health/ready    # Readiness check
```

## Scaling Checklist

1. **Resource Monitoring**
   - [ ] Configure CPU alerts
   - [ ] Set memory usage alerts
   - [ ] Monitor network latency
   - [ ] Track error rates

2. **Performance Tuning**
   - [ ] Optimize worker count
   - [ ] Configure request limits
   - [ ] Enable caching
   - [ ] Set appropriate timeouts

3. **Load Testing**
   - [ ] Test concurrent requests
   - [ ] Measure memory usage
   - [ ] Monitor response times
   - [ ] Verify error handling

## Best Practices

1. **Resource Management**
    - Start with conservative limits
    - Monitor actual usage
    - Scale gradually
    - Implement proper cleanup

2. **Error Handling**
    - Set appropriate timeouts
    - Implement retry logic
    - Monitor failure rates
    - Log errors comprehensively

3. **Performance**
    - Use efficient formats
    - Optimize wait strategies
    - Enable caching
    - Monitor response times

For detailed monitoring configuration, see the [Monitoring Guide](monitoring.md).