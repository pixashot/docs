---
excerpt: "Comprehensive overview of deployment options for Pixashot, including cloud platforms, self-hosted solutions, and scaling considerations."
published_at: true
---

# Deployment

Pixashot can be deployed on various cloud platforms or self-hosted environments. This section provides an overview of deployment options and links to detailed documentation for each platform.

## Google Cloud Run

Google Cloud Run is an excellent option for deploying Pixashot, offering easy scaling and a generous free tier. For a comprehensive, step-by-step guide to deploying Pixashot on Cloud Run, see our [detailed Cloud Run deployment guide](cloud-run-deployment.md).

```bash
# Deploy to Cloud Run with optimal settings
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 2Gi \
  --cpu 2 \
  --set-env-vars="AUTH_TOKEN=your_secret_token,\
USE_POPUP_BLOCKER=true,\
USE_COOKIE_BLOCKER=true,\
WORKERS=4,\
MAX_REQUESTS=1000,\
KEEP_ALIVE=300"
```

For more general information about Cloud Run deployment, refer to:
- [Deploying container images on Cloud Run](https://cloud.google.com/run/docs/deploying)
- [Setting up continuous deployment](https://cloud.google.com/run/docs/continuous-deployment)
- [Managing secrets in Cloud Run](https://cloud.google.com/run/docs/configuring/secrets)

## AWS

Amazon Web Services offers multiple options for deploying containerized applications:

- [AWS ECS (Elastic Container Service)](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [AWS EKS (Elastic Kubernetes Service)](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [AWS App Runner](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html)

For serverless deployments, consider:
- [AWS Lambda with Container Images](https://docs.aws.amazon.com/lambda/latest/dg/lambda-images.html)

## Azure

Microsoft Azure provides several services for container deployment:

- [Azure Container Instances](https://docs.microsoft.com/en-us/azure/container-instances/)
- [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/)
- [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/configure-custom-container?pivots=container-linux)

## DigitalOcean Apps

DigitalOcean Apps platform offers a simple way to deploy containerized applications:

- [Deploying a Docker Image to DigitalOcean App Platform](https://docs.digitalocean.com/products/app-platform/how-to/deploy-a-docker-image/)
- [App Platform Overview](https://docs.digitalocean.com/products/app-platform/)

## Self-hosted Options

For self-hosted deployments, consider the following options:

- [Docker Compose](https://docs.docker.com/compose/): Ideal for single-server deployments
- [Kubernetes](https://kubernetes.io/docs/home/): For more complex, scalable deployments
- [Podman](https://podman.io/): An alternative to Docker for rootless containers

## Docker Deployment

Pixashot is available as a Docker image, making it easy to deploy in any environment that supports Docker:

```bash
# Basic deployment
docker pull gpriday/pixashot:latest
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest

# Advanced deployment with resource limits
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  -e WORKERS=4 \
  -e MAX_REQUESTS=1000 \
  -e KEEP_ALIVE=300 \
  -e RATE_LIMIT_ENABLED=true \
  -e RATE_LIMIT_CAPTURE="1 per second" \
  --memory=2g \
  gpriday/pixashot:latest
```

For more details on Docker deployment, refer to:
- [Docker documentation](https://docs.docker.com/)
- [Docker Compose documentation](https://docs.docker.com/compose/)

## Environment Variables

Key environment variables for configuring your deployment:

### Required Settings
- `AUTH_TOKEN`: Authentication token for API requests (required)
- `PORT`: Server port (default: 8080)

### Worker Configuration
- `WORKERS`: Number of worker processes (default: 4)
- `MAX_REQUESTS`: Maximum requests per worker before restart (default: 1000)
- `KEEP_ALIVE`: Keep-alive timeout in seconds (default: 300)

### Browser Extensions
- `USE_POPUP_BLOCKER`: Enable/disable popup blocker extension (default: true)
- `USE_COOKIE_BLOCKER`: Enable/disable cookie consent blocker extension (default: true)

### Rate Limiting
- `RATE_LIMIT_ENABLED`: Enable/disable rate limiting (default: false)
- `RATE_LIMIT_CAPTURE`: Rate limit for capture endpoint (default: "1 per second")
- `RATE_LIMIT_SIGNED`: Rate limit for signed URLs (default: "5 per second")

### Proxy Configuration (Optional)
- `PROXY_SERVER`: Proxy server address
- `PROXY_PORT`: Proxy server port
- `PROXY_USERNAME`: Proxy server username
- `PROXY_PASSWORD`: Proxy server password

### Caching (Optional)
- `CACHE_MAX_SIZE`: Maximum size for response caching (default: 0, disabled)

## Architecture Considerations

Pixashot uses a single-context architecture where all requests share the same browser context. This design:
- Improves memory efficiency
- Reduces cold start times
- Provides better resource utilization

Key considerations for this architecture:

1. **Browser Extensions**:
   - Must be configured via environment variables at startup
   - Cannot be changed per-request
   - Apply to all requests in the context

2. **Resource Management**:
   - Memory is shared across all requests
   - Single browser instance reduces overhead
   - Worker processes handle request distribution

## Scaling Considerations

When deploying Pixashot, consider the following for optimal performance:

1. **Resource Allocation**:
   - CPU: Minimum 2 cores recommended
   - Memory: Minimum 2GB per instance
   - Disk: At least 1GB for browser data

2. **Concurrency Settings**:
   - Adjust `WORKERS` based on available CPU cores
   - Set `MAX_REQUESTS` to prevent memory leaks
   - Configure `KEEP_ALIVE` for your usage patterns

3. **Load Balancing**:
   - Use platform-native load balancing when available
   - Consider sticky sessions for better context reuse
   - Implement health checks for proper instance rotation

4. **Monitoring**:
   - Track memory usage across workers
   - Monitor browser context health
   - Set up alerting for error rates and response times

## Health Checks

The service provides three health check endpoints:

- `/health/live`: Basic liveness check
- `/health/ready`: Readiness check including browser context status
- `/health`: Detailed health status including memory usage and request statistics

Configure these in your platform's health check settings:

```bash
# Example health check configuration for Cloud Run
gcloud run services update pixashot \
  --set-health-checks-path=/health/live \
  --min-instances=1
```

Remember to always test your deployment thoroughly in a staging environment before going live. Each platform has its own set of best practices and optimization techniques, so be sure to consult the specific documentation for your chosen deployment method.