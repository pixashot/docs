---
title: Rate Limiting
excerpt: Comprehensive guide to Pixashot's rate limiting system, including configuration, monitoring, and best practices for handling rate limits.
meta:
    nav_order: 74
---

# Rate Limiting

Pixashot implements rate limiting to ensure fair usage and system stability. This guide covers rate limit configuration, monitoring, and handling.

## Configuration

Rate limiting is configured through environment variables:

```bash
# Enable rate limiting
RATE_LIMIT_ENABLED=true

# Set limits
RATE_LIMIT_CAPTURE="5 per second"     # Main capture endpoint
RATE_LIMIT_SIGNED="10 per second"     # Signed URL requests
```

## Rate Limit Headers

All responses include rate limit information:

```http
X-RateLimit-Limit: 5           # Requests allowed per window
X-RateLimit-Remaining: 4       # Remaining requests
X-RateLimit-Reset: 1635724800  # Timestamp when limit resets
```

## Rate Limit Response

When limits are exceeded:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 5

{
    "status": "error",
    "message": "Rate limit exceeded",
    "error_type": "RateLimitError",
    "retry_after": 5
}
```

## Client Implementation

### Example with Retry Logic

```javascript
async function captureWithRetry(options, maxAttempts = 3) {
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            const response = await fetch('/capture', {
                method: 'POST',
                headers: {
                    'Authorization': `Bearer ${AUTH_TOKEN}`,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(options)
            });

            if (response.status === 429) {
                const retryAfter = parseInt(response.headers.get('Retry-After') || '5');
                await new Promise(r => setTimeout(r, retryAfter * 1000));
                continue;
            }

            if (!response.ok) throw new Error(`HTTP ${response.status}`);
            return response;

        } catch (error) {
            if (attempt === maxAttempts) throw error;
            await new Promise(r => setTimeout(r, Math.pow(2, attempt) * 1000));
        }
    }
}
```

## Rate Limit Strategies

### 1. Queue Management
```javascript
class RequestQueue {
    constructor(maxConcurrent = 1) {
        this.queue = [];
        this.running = 0;
        this.maxConcurrent = maxConcurrent;
    }

    async add(request) {
        if (this.running >= this.maxConcurrent) {
            await new Promise(resolve => this.queue.push(resolve));
        }
        
        this.running++;
        try {
            return await request();
        } finally {
            this.running--;
            if (this.queue.length > 0) {
                const next = this.queue.shift();
                next();
            }
        }
    }
}
```

### 2. Token Bucket Implementation
```javascript
class RateLimiter {
    constructor(rate, capacity) {
        this.tokens = capacity;
        this.capacity = capacity;
        this.rate = rate;
        this.lastRefill = Date.now();
    }

    async acquire() {
        await this.refill();
        if (this.tokens < 1) {
            throw new Error('Rate limit exceeded');
        }
        this.tokens--;
        return true;
    }

    async refill() {
        const now = Date.now();
        const elapsed = (now - this.lastRefill) / 1000;
        this.tokens = Math.min(
            this.capacity,
            this.tokens + elapsed * this.rate
        );
        this.lastRefill = now;
    }
}
```

## Best Practices

### 1. Monitor Rate Limits
```javascript
function getRateLimitInfo(response) {
    return {
        limit: parseInt(response.headers.get('X-RateLimit-Limit')),
        remaining: parseInt(response.headers.get('X-RateLimit-Remaining')),
        reset: parseInt(response.headers.get('X-RateLimit-Reset'))
    };
}
```

### 2. Implement Backoff Strategy
```javascript
async function exponentialBackoff(attempt) {
    const delay = Math.min(1000 * Math.pow(2, attempt), 30000);
    await new Promise(r => setTimeout(r, delay));
}
```

### 3. Request Batching
```javascript
async function batchRequests(urls, batchSize = 5) {
    const results = [];
    for (let i = 0; i < urls.length; i += batchSize) {
        const batch = urls.slice(i, i + batchSize);
        const batchResults = await Promise.all(
            batch.map(url => captureWithRetry({ url }))
        );
        results.push(...batchResults);
        await new Promise(r => setTimeout(r, 1000)); // Rate control
    }
    return results;
}
```

## Production Considerations

1. **Default Limits**
    - Capture endpoint: 5 requests/second
    - Signed URLs: 10 requests/second
    - Adjust based on your service tier

2. **Monitoring**
    - Track rate limit errors
    - Monitor usage patterns
    - Set up alerts for high rates

3. **Client Configuration**
    - Implement retry logic
    - Use request queuing
    - Monitor rate limit headers

4. **Performance Tips**
    - Batch related requests
    - Implement client-side caching
    - Use appropriate timeout values