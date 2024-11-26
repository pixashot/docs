---
title: Docker Deployment
excerpt: Guide to running Pixashot using Docker, including container configuration, resource management, and production deployment strategies.
meta:
    nav_order: 152
---

# Docker Deployment Guide

This guide covers deploying Pixashot using Docker, including configuration, resource management, and production best practices.

## Quick Start

```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest
```

## Image Structure

Pixashot's Docker image is built on the official Playwright Python image with additional optimizations:

```dockerfile
# Base image with Playwright
FROM mcr.microsoft.com/playwright/python:v1.47.0

# Environment configuration
ENV PORT=8080 \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONPATH=/app \
    PIP_NO_CACHE_DIR=1 \
    PIP_ROOT_USER_ACTION=ignore

# Non-root user setup
RUN useradd -m appuser && \
    mkdir -p /tmp/screenshots /app/data /app/logs && \
    chown -R appuser:appuser /app /tmp/screenshots /app/data /app/logs

# Install requirements
COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Install Playwright browser
RUN playwright install --with-deps chromium

# Copy application code
COPY src/ /app
COPY entry.sh .

USER appuser

EXPOSE ${PORT}
CMD ["./entry.sh"]
```

## Environment Variables

### Required Settings
```bash
# Core Configuration
AUTH_TOKEN=                  # Authentication token
PORT=8080                    # Server port (default: 8080)

# Worker Configuration
WORKERS=4                    # Number of worker processes
MAX_REQUESTS=1000           # Maximum requests per worker
KEEP_ALIVE=300             # Keep-alive timeout in seconds
```

### Optional Settings
```bash
# Feature Toggles
USE_POPUP_BLOCKER=true      # Enable popup blocking
USE_COOKIE_BLOCKER=true     # Enable cookie consent blocking

# Performance
RATE_LIMIT_ENABLED=false    # Enable rate limiting
RATE_LIMIT_CAPTURE="1/s"    # Capture endpoint rate limit
CACHE_MAX_SIZE=1000        # Response cache size

# Proxy Configuration
PROXY_SERVER=               # Proxy server address
PROXY_PORT=                 # Proxy server port
PROXY_USERNAME=             # Proxy authentication
PROXY_PASSWORD=             # Proxy authentication
```

## Resource Allocation

### Memory Management

```bash
# Minimum viable configuration
docker run -m 2g --memory-swap 2g \
  gpriday/pixashot:latest

# Production configuration
docker run -m 4g --memory-swap 4g \
  --memory-reservation 2g \
  --kernel-memory 512m \
  gpriday/pixashot:latest
```

### CPU Allocation

```bash
# Basic configuration
docker run --cpus=1 \
  gpriday/pixashot:latest

# Production with CPU pinning
docker run --cpus=2 \
  --cpuset-cpus="0,1" \
  gpriday/pixashot:latest
```

## Volume Mounting

### Persistent Storage

```bash
# Mount cache directory
docker run -v pixashot_cache:/app/cache \
  gpriday/pixashot:latest

# Mount logs
docker run -v pixashot_logs:/app/logs \
  gpriday/pixashot:latest
```

### Directory Structure
```
/app/
├── cache/              # Cache storage
├── logs/              # Application logs
└── data/              # Temporary data
```

## Docker Compose

### Basic Setup

```yaml
# docker-compose.yml
version: '3.8'

services:
  pixashot:
    image: gpriday/pixashot:latest
    ports:
      - "8080:8080"
    environment:
      - AUTH_TOKEN=your_secret_token
      - WORKERS=4
      - MAX_REQUESTS=1000
      - USE_POPUP_BLOCKER=true
      - USE_COOKIE_BLOCKER=true
    volumes:
      - pixashot_cache:/app/cache
      - pixashot_logs:/app/logs
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

volumes:
  pixashot_cache:
  pixashot_logs:
```

### Production Setup

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  pixashot:
    image: gpriday/pixashot:latest
    ports:
      - "8080:8080"
    environment:
      - AUTH_TOKEN=${AUTH_TOKEN}
      - WORKERS=8
      - MAX_REQUESTS=1000
      - USE_POPUP_BLOCKER=true
      - USE_COOKIE_BLOCKER=true
      - RATE_LIMIT_ENABLED=true
      - RATE_LIMIT_CAPTURE=5 per second
      - CACHE_MAX_SIZE=1000
    volumes:
      - pixashot_cache:/app/cache
      - pixashot_logs:/app/logs
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: any
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health/live"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

volumes:
  pixashot_cache:
    driver: local
  pixashot_logs:
    driver: local
```

## Health Checks

### Docker Health Check

```bash
# Basic health check
docker run \
  --health-cmd="curl -f http://localhost:8080/health/live || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=40s \
  gpriday/pixashot:latest
```

### Available Health Endpoints

```bash
# Liveness check
curl http://localhost:8080/health/live

# Readiness check
curl http://localhost:8080/health/ready

# Detailed health status
curl http://localhost:8080/health
```

## Multi-stage Build

```dockerfile
# Build stage
FROM python:3.11-slim as builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Runtime stage
FROM mcr.microsoft.com/playwright/python:v1.47.0

ENV PATH="/home/appuser/.local/bin:${PATH}"

# Copy Python packages
COPY --from=builder /root/.local /home/appuser/.local

# Non-root user setup
RUN useradd -m appuser && \
    mkdir -p /tmp/screenshots /app/data /app/logs && \
    chown -R appuser:appuser /app /tmp/screenshots /app/data /app/logs

# Install Playwright browser
RUN playwright install --with-deps chromium

COPY src/ /app
COPY entry.sh .

USER appuser
EXPOSE 8080
CMD ["./entry.sh"]
```

## Troubleshooting

### Common Issues

1. **Memory Exceeded**
```bash
# Check memory usage
docker stats pixashot

# Increase memory limit
docker run -m 4g --memory-swap 4g gpriday/pixashot:latest
```

2. **Container Crashes**
```bash
# View logs
docker logs pixashot

# Enable debug logging
docker run -e LOG_LEVEL=debug gpriday/pixashot:latest
```

3. **Performance Issues**
```bash
# Check resource usage
docker stats

# Monitor processes inside container
docker exec -it pixashot top
```

4. **Network Issues**
```bash
# Check container networking
docker network inspect bridge

# Use host networking (if needed)
docker run --network host gpriday/pixashot:latest
```

### Diagnostic Commands

```bash
# Container logs
docker logs pixashot

# Container processes
docker top pixashot

# Resource usage
docker stats pixashot

# Container inspection
docker inspect pixashot

# Execute commands inside container
docker exec -it pixashot /bin/bash
```

## Next Steps

- Set up [monitoring](cloud-run.md)
- Configure [logging](logging.md)
- Review [security best practices](../security/index.md)
- Implement [rate limiting](../api-reference/rate-limiting.md)

For more information, refer to the [API Reference](../api-reference/index.md) and [Feature Documentation](../api-reference/index.md).