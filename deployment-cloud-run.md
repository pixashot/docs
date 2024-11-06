# Deploying Pixashot on Google Cloud Run

This guide provides step-by-step instructions for deploying Pixashot on Google Cloud Run, a fully managed platform for containerized applications.

## Prerequisites

Before you begin, ensure you have:
- A Google Cloud Platform account with billing enabled
- Google Cloud SDK (gcloud CLI) installed
- Docker installed on your local machine (only needed if building custom images)

## Quick Start

For the fastest deployment using our pre-built image:

```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 2Gi \
  --cpu 2 \
  --port 8080 \
  --set-env-vars="CLOUD_RUN=true,AUTH_TOKEN=your_secret_token"
```

## Detailed Deployment Steps

### 1. Initial Setup

```bash
# Login to Google Cloud
gcloud auth login

# Set your project ID
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable run.googleapis.com
gcloud services enable containerregistry.googleapis.com
```

### 2. Configuration Options

Pixashot can be configured using environment variables:

```bash
# Required settings
AUTH_TOKEN=your_secret_token     # Authentication token for API requests
CLOUD_RUN=true                   # Enable Cloud Run optimizations

# Optional settings (with defaults)
WORKERS=4                        # Number of worker processes
MEMORY_LIMIT=2048               # Memory limit in MB
TIMEOUT=300                     # Request timeout in seconds
MAX_REQUESTS=1000               # Maximum requests per worker
```

### 3. Deployment Methods

#### Option 1: Using Google Cloud Console

1. Go to Cloud Run in Google Cloud Console
2. Click "Create Service"
3. Select "Deploy one revision from an existing container image"
4. Enter `gpriday/pixashot:latest` as the container image
5. Configure the following settings:
   - Memory: 2GB
   - CPU: 2
   - Request timeout: 300 seconds
   - Maximum concurrent requests: 80
   - Environment variables (as listed above)
   - Port: 8080

#### Option 2: Using Command Line

```bash
# Basic deployment
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 2Gi \
  --cpu 2 \
  --port 8080 \
  --set-env-vars="CLOUD_RUN=true,AUTH_TOKEN=your_secret_token"

# Advanced deployment with all options
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --allow-unauthenticated \
  --region us-central1 \
  --memory 2Gi \
  --cpu 2 \
  --port 8080 \
  --timeout 300 \
  --concurrency 80 \
  --min-instances 0 \
  --max-instances 10 \
  --set-env-vars="CLOUD_RUN=true,AUTH_TOKEN=your_secret_token,WORKERS=4,MEMORY_LIMIT=2048,MAX_REQUESTS=1000"
```

### 4. Resource Configuration

Recommended resource settings for different workloads:

```bash
# Light workload (1-10 requests/minute)
--memory 1Gi --cpu 1 --concurrency 40

# Medium workload (10-50 requests/minute)
--memory 2Gi --cpu 2 --concurrency 80

# Heavy workload (50+ requests/minute)
--memory 4Gi --cpu 4 --concurrency 100
```

### 5. Continuous Deployment

Set up continuous deployment using Cloud Build:

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'pixashot'
      - '--image'
      - 'gpriday/pixashot:latest'
      - '--platform'
      - 'managed'
      - '--region'
      - 'us-central1'
      - '--memory'
      - '2Gi'
      - '--cpu'
      - '2'
      - '--set-env-vars'
      - 'CLOUD_RUN=true,AUTH_TOKEN=${_AUTH_TOKEN}'
```

### 6. Monitoring and Health Checks

Configure health checks in Cloud Run:

1. Liveness Check:
   - Path: `/health/live`
   - Initial delay: 10s
   - Interval: 30s
   - Timeout: 5s
   - Failure threshold: 3

2. Readiness Check:
   - Path: `/health/ready`
   - Initial delay: 15s
   - Interval: 30s
   - Timeout: 5s
   - Failure threshold: 3

### 7. Cost Optimization

Tips for optimizing costs:

1. Set minimum instances to 0 for dev/staging environments
2. Configure maximum instances based on expected load
3. Use the free tier efficiently (2 million requests/month)
4. Monitor and adjust resource allocation based on usage

### 8. Security Best Practices

1. **Authentication**:
   - Store AUTH_TOKEN in Secret Manager:
   ```bash
   # Create secret
   echo -n "your_secret_token" | gcloud secrets create pixashot-auth-token --data-file=-
   
   # Use in deployment
   gcloud run deploy pixashot \
     ... \
     --set-secrets="AUTH_TOKEN=pixashot-auth-token:latest"
   ```

2. **Network Security**:
   - Enable Cloud Run VPC connector if needed
   - Configure ingress settings appropriately

### 9. Troubleshooting

Common issues and solutions:

1. **Cold Start Issues**:
   - Set minimum instances > 0
   - Optimize container startup time

2. **Memory Issues**:
   - Increase memory allocation
   - Monitor memory usage patterns

3. **Timeout Issues**:
   - Adjust timeout settings
   - Optimize request handling

## Testing the Deployment

After deployment, test your instance:

```bash
# Get the service URL
SERVICE_URL=$(gcloud run services describe pixashot --format='value(status.url)')

# Test the health endpoint
curl $SERVICE_URL/health/live

# Test screenshot capture
curl -X POST $SERVICE_URL/capture \
  -H "Authorization: Bearer your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com","format":"png"}'
```

Remember that Cloud Run automatically handles scaling based on traffic, and you only pay for the resources used during request processing. Monitor your service's performance and costs regularly to optimize your deployment.