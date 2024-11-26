---
title: Installing Pixashot
excerpt: Step-by-step installation guide for deploying Pixashot using Docker or Google Cloud Run.
meta:
    nav_order: 22
---

# Installing Pixashot

This guide covers installing Pixashot using Docker (recommended for development and self-hosting) or Google Cloud Run (recommended for production). Both methods use our official Docker image, ensuring consistent behavior across environments.

## Docker Installation

### Prerequisites
- Docker 20.10.0 or higher installed
- 2GB minimum available memory
- Outbound internet access for pulling images

### Basic Docker Setup

1. Pull the latest Pixashot image:
```bash
docker pull gpriday/pixashot:latest
```

2. Create a secure authentication token:
```bash
export AUTH_TOKEN=$(openssl rand -hex 32)
```

3. Run the container:
```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN=$AUTH_TOKEN \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  gpriday/pixashot:latest
```

4. Verify the installation:
```bash
# Check health endpoint
curl http://localhost:8080/health

# Test screenshot capture
curl -X POST http://localhost:8080/capture \
  -H "Authorization: Bearer $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png"
  }'
```

### Advanced Docker Configuration

For production Docker deployments, use these additional settings:

```bash
docker run -p 8080:8080 \
  -e AUTH_TOKEN=$AUTH_TOKEN \
  -e USE_POPUP_BLOCKER=true \
  -e USE_COOKIE_BLOCKER=true \
  -e WORKERS=4 \
  -e MAX_REQUESTS=1000 \
  -e KEEP_ALIVE=300 \
  -e RATE_LIMIT_ENABLED=true \
  -e RATE_LIMIT_CAPTURE="1 per second" \
  -e CACHE_MAX_SIZE=1000 \
  --memory=2g \
  --cpu-shares=2048 \
  gpriday/pixashot:latest
```

## Google Cloud Run Installation

### Prerequisites
- Google Cloud SDK installed
- Project with billing enabled
- Required APIs enabled
- Proper IAM permissions

### Initial Setup

1. Install the Google Cloud SDK if you haven't already:
```bash
# Visit: https://cloud.google.com/sdk/docs/install
# After installation:
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
```

2. Enable required APIs:
```bash
gcloud services enable run.googleapis.com
gcloud services enable secretmanager.googleapis.com
```

### Basic Cloud Run Deployment

1. Generate a secure authentication token:
```bash
export AUTH_TOKEN=$(openssl rand -hex 32)
```

2. Deploy to Cloud Run:
```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 2Gi \
  --cpu 1 \
  --port 8080 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=$AUTH_TOKEN,\
USE_POPUP_BLOCKER=true,\
USE_COOKIE_BLOCKER=true,\
WORKERS=4,\
MAX_REQUESTS=1000,\
KEEP_ALIVE=300"
```

3. Get your service URL:
```bash
export SERVICE_URL=$(gcloud run services describe pixashot --format='value(status.url)')
```

4. Test the deployment:
```bash
curl -X POST $SERVICE_URL/capture \
  -H "Authorization: Bearer $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png"
  }'
```

### Production Cloud Run Configuration

For production deployments, use these enhanced settings:

```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 4Gi \
  --cpu 2 \
  --port 8080 \
  --timeout 300 \
  --concurrency 80 \
  --min-instances 1 \
  --max-instances 10 \
  --set-env-vars="AUTH_TOKEN=$AUTH_TOKEN,\
USE_POPUP_BLOCKER=true,\
USE_COOKIE_BLOCKER=true,\
WORKERS=8,\
MAX_REQUESTS=1000,\
KEEP_ALIVE=300,\
RATE_LIMIT_ENABLED=true,\
RATE_LIMIT_CAPTURE=5 per second,\
CACHE_MAX_SIZE=1000"
```

## Verifying Your Installation

Regardless of deployment method, verify your installation using these health checks:

1. Basic Health Check:
```bash
curl $SERVICE_URL/health
```

2. Readiness Check:
```bash
curl $SERVICE_URL/health/ready
```

3. Test Screenshot Capture:
```bash
curl -X POST $SERVICE_URL/capture \
  -H "Authorization: Bearer $AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true,
    "window_width": 1280,
    "window_height": 720
  }'
```

## Post-Installation Steps

After successful installation:

1. Store your AUTH_TOKEN securely
2. Configure monitoring
3. Set up health check alerts
4. Review rate limits
5. Test different capture scenarios

### For Docker Installations
- Configure container restart policies
- Set up log rotation
- Monitor container health
- Configure backups if needed

### For Cloud Run Installations
- Set up Cloud Monitoring
- Configure error reporting
- Set up logging exports
- Review scaling settings

## Common Installation Issues

### Docker Issues
- **Memory Errors**: Increase container memory limit
- **Network Errors**: Check outbound connectivity
- **Permission Errors**: Verify container user permissions
- **Resource Exhaustion**: Monitor and adjust resource limits

### Cloud Run Issues
- **Cold Start Issues**: Configure minimum instances
- **Timeout Errors**: Adjust timeout settings
- **Memory Errors**: Increase memory allocation
- **Scaling Issues**: Review con