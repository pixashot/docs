---
excerpt: "Step-by-step guide to deploying Pixashot on Google Cloud Run, including initial setup, configuration options, monitoring, and best practices for production deployments."
published_at: true
---

# Deploying Pixashot on Google Cloud Run

This guide provides step-by-step instructions for deploying Pixashot on Google Cloud Run, a fully managed platform for containerized applications.

## Initial Setup (Required)

Before deploying Pixashot, you must complete these one-time setup steps:

```bash
# 1. Install Google Cloud SDK if you haven't already
# Visit: https://cloud.google.com/sdk/docs/install

# 2. Login to Google Cloud
gcloud auth login

# 3. Set your project ID
gcloud config set project YOUR_PROJECT_ID

# 4. Enable required APIs
gcloud services enable run.googleapis.com
gcloud services enable secretmanager.googleapis.com
```

## Quick Start

Pixashot is available as a pre-built Docker image on Docker Hub, making deployment quick and easy. Once you've completed the initial setup above, you can deploy directly from Docker Hub:

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
  --set-env-vars="AUTH_TOKEN=your_secret_token,\
USE_POPUP_BLOCKER=true,\
USE_COOKIE_BLOCKER=true,\
WORKERS=4,\
MAX_REQUESTS=1000,\
KEEP_ALIVE=300"
```

⚠️ **Important Notes**:
- Replace `your_secret_token` with a secure authentication token. This token will be required for all API requests to your Pixashot instance.
- The image `gpriday/pixashot:latest` will be automatically pulled from Docker Hub. No additional registry configuration is required.

After deployment, you can test your instance:
```bash
# Get your service URL
SERVICE_URL=$(gcloud run services describe pixashot --format='value(status.url)')

# Test with a simple screenshot request
curl -X POST $SERVICE_URL/capture \
  -H "Authorization: Bearer your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com","format":"png"}'
```

## Environment Variables

### Required Settings
- `AUTH_TOKEN`: Authentication token for API requests
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

### Optional Settings
- `CACHE_MAX_SIZE`: Maximum size for response caching (default: 0, disabled)
- `PROXY_SERVER`, `PROXY_PORT`, `PROXY_USERNAME`, `PROXY_PASSWORD`: Proxy configuration

## Deployment Methods

### Option 1: Using Google Cloud Console

1. Go to Cloud Run in Google Cloud Console
2. Click "Create Service"
3. Select "Deploy one revision from an existing container image"
4. Enter `gpriday/pixashot:latest` as the container image
5. Configure the service:
   - Memory: 2GB (minimum recommended)
   - CPU: 1 (recommended)
   - Maximum requests per container: 80
   - Request timeout: 300 seconds
   - Environment variables (as listed above)
   - Port: 8080

### Option 2: Advanced Command Line Deployment

```bash
# Advanced deployment with all options
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 4Gi \
  --cpu 2 \
  --port 8080 \
  --timeout 300 \
  --concurrency 100 \
  --min-instances 1 \
  --max-instances 10 \
  --set-env-vars="AUTH_TOKEN=your_secret_token,\
USE_POPUP_BLOCKER=true,\
USE_COOKIE_BLOCKER=true,\
WORKERS=8,\
MAX_REQUESTS=1000,\
KEEP_ALIVE=300,\
RATE_LIMIT_ENABLED=true,\
RATE_LIMIT_CAPTURE=5 per second,\
CACHE_MAX_SIZE=1000"
```

## Resource Configuration

Recommended resource settings for different workloads:

```bash
# Light workload (1-10 requests/minute)
--memory 2Gi --cpu 1 --concurrency 40 --min-instances 0 --max-instances 2

# Medium workload (10-50 requests/minute)
--memory 2Gi --cpu 2 --concurrency 80 --min-instances 1 --max-instances 5

# Heavy workload (50+ requests/minute)
--memory 4Gi --cpu 4 --concurrency 100 --min-instances 2 --max-instances 10
```

## Continuous Deployment

Example Cloud Build configuration:

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - '${_SERVICE_NAME}'
      - '--image'
      - 'gpriday/pixashot:latest'
      - '--platform'
      - 'managed'
      - '--region'
      - '${_REGION}'
      - '--memory'
      - '${_MEMORY}'
      - '--cpu'
      - '${_CPU}'
      - '--timeout'
      - '${_TIMEOUT}'
      - '--set-env-vars'
      - 'AUTH_TOKEN=${_AUTH_TOKEN},USE_POPUP_BLOCKER=${_USE_POPUP_BLOCKER},USE_COOKIE_BLOCKER=${_USE_COOKIE_BLOCKER},WORKERS=${_WORKERS},MAX_REQUESTS=${_MAX_REQUESTS},KEEP_ALIVE=${_KEEP_ALIVE}'

substitutions:
  _SERVICE_NAME: pixashot
  _REGION: us-central1
  _MEMORY: 2Gi
  _CPU: "2"
  _TIMEOUT: "300"
  _USE_POPUP_BLOCKER: "true"
  _USE_COOKIE_BLOCKER: "true"
  _WORKERS: "4"
  _MAX_REQUESTS: "1000"
  _KEEP_ALIVE: "300"
```

## Health Checks and Monitoring

### Configure Health Checks

1. Liveness Check:
```bash
gcloud run services update pixashot \
  --set-liveness-check-path=/health/live \
  --set-liveness-initial-delay=10s \
  --set-liveness-timeout=5s \
  --set-liveness-failure-threshold=3
```

2. Startup Check:
```bash
gcloud run services update pixashot \
  --set-startup-check-path=/health/ready \
  --set-startup-initial-delay=15s \
  --set-startup-timeout=5s \
  --set-startup-failure-threshold=3
```

### Set Up Monitoring

```bash
# Create memory usage alert
gcloud beta monitoring alerts policies create \
  --display-name="Pixashot High Memory Usage" \
  --condition="resource.type = \"cloud_run_revision\" AND metric.type = \"run.googleapis.com/container/memory/utilization\" > 0.8" \
  --duration=300s

# View logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=pixashot"
```

## Security Best Practices

### Secure Authentication Token

```bash
# Store auth token in Secret Manager
echo -n "your_secret_token" | \
  gcloud secrets create pixashot-auth-token --data-file=- --replication-policy="automatic"

# Use secret in deployment
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --set-secrets="/AUTH_TOKEN=pixashot-auth-token:latest" \
  ... # other options
```

### Network Security

```bash
# Configure ingress settings
gcloud run services update pixashot \
  --ingress internal-and-cloud-load-balancing
```

## Troubleshooting

### Common Issues and Solutions

1. **Cold Start Issues**
   - Increase minimum instances
   - Check startup logs: `gcloud logging read "resource.type=cloud_run_revision AND severity>=ERROR"`
   - Monitor startup times in Cloud Run metrics

2. **Memory Issues**
   - Review memory usage: `gcloud monitoring metrics list | grep memory`
   - Increase memory allocation if needed
   - Check for memory leaks in logs

3. **Browser Context Issues**
   - Monitor browser health in logs
   - Check error patterns
   - Verify extension settings

## Testing and Verification

```bash
# Get the service URL
SERVICE_URL=$(gcloud run services describe pixashot --format='value(status.url)')

# Test health endpoints
curl $SERVICE_URL/health/live
curl $SERVICE_URL/health/ready
curl $SERVICE_URL/health

# Test screenshot capture
curl -X POST $SERVICE_URL/capture \
  -H "Authorization: Bearer your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true,
    "window_width": 1280,
    "window_height": 720
  }'
```

## Cost Optimization

1. **Instance Management**
   - Use minimum instances of 0 for non-critical services
   - Set appropriate maximum instances to control costs
   - Use scheduled scaling for predictable loads

2. **Resource Allocation**
   - Start with 2Gi memory and 1 CPU
   - Monitor usage patterns
   - Adjust based on actual needs

3. **Request Handling**
   - Configure appropriate timeouts
   - Enable caching when possible
   - Implement rate limiting

Remember that Cloud Run automatically handles scaling based on traffic, and you only pay for the resources used during request processing. Monitor your service's performance and costs regularly to optimize your deployment.