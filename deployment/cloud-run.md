---
title: Google Cloud Run Deployment
excerpt: Comprehensive guide to deploying Pixashot on Google Cloud Run, including setup, configuration, monitoring, and production best practices.
meta:
    nav_order: 61
---

# Google Cloud Run Deployment

This guide provides comprehensive instructions for deploying Pixashot on Google Cloud Run, including initial setup, configuration, monitoring, and production best practices.

## Prerequisites

Before deploying Pixashot, ensure you have:
- Google Cloud account
- Google Cloud SDK installed
- Docker installed (optional for local testing)
- Project with billing enabled

## Initial Setup

1. **Install Google Cloud SDK**:
```bash
# Visit: https://cloud.google.com/sdk/docs/install
# After installation:
gcloud auth login
gcloud config set project YOUR_PROJECT_ID
```

2. **Enable Required APIs**:
```bash
gcloud services enable run.googleapis.com
gcloud services enable secretmanager.googleapis.com
gcloud services enable cloudmonitoring.googleapis.com
gcloud services enable cloudbuild.googleapis.com
```

## Basic Deployment

Deploy Pixashot with default settings:

```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --region us-central1 \
  --memory 2Gi \
  --cpu 1 \
  --timeout 300 \
  --set-env-vars="AUTH_TOKEN=your_secret_token"
```

## Environment Configuration

### Required Variables
```bash
# Core Settings
AUTH_TOKEN=           # Authentication token
PORT=8080             # Server port

# Worker Configuration
WORKERS=4             # Number of worker processes
MAX_REQUESTS=1000     # Requests per worker before restart
KEEP_ALIVE=300        # Keep-alive timeout (seconds)
```

### Optional Variables
```bash
# Feature Toggles
USE_POPUP_BLOCKER=true      # Enable popup blocking
USE_COOKIE_BLOCKER=true     # Enable cookie consent blocking

# Performance
RATE_LIMIT_ENABLED=true     # Enable rate limiting
RATE_LIMIT_CAPTURE="1/s"    # Capture endpoint limit
CACHE_MAX_SIZE=1000         # Response cache size
```

## Advanced Deployment Options

### Production Configuration
```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --platform managed \
  --region us-central1 \
  --memory 4Gi \
  --cpu 2 \
  --timeout 300 \
  --concurrency 80 \
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

## Security Configuration

### Secret Management

1. **Create Secret**:
```bash
echo -n "your_secret_token" | \
  gcloud secrets create pixashot-auth-token \
  --data-file=- \
  --replication-policy="automatic"
```

2. **Use Secret in Deployment**:
```bash
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --set-secrets="/AUTH_TOKEN=pixashot-auth-token:latest"
```

### IAM Configuration

```bash
# Allow public access
gcloud run services add-iam-policy-binding pixashot \
  --region=us-central1 \
  --member="allUsers" \
  --role="roles/run.invoker"

# Restrict to authenticated users
gcloud run services add-iam-policy-binding pixashot \
  --region=us-central1 \
  --member="user:user@example.com" \
  --role="roles/run.invoker"
```

## Monitoring Setup

### Health Checks

1. **Configure Liveness Check**:
```bash
gcloud run services update pixashot \
  --set-liveness-check-path=/health/live \
  --set-liveness-initial-delay=10s \
  --set-liveness-timeout=5s \
  --set-liveness-failure-threshold=3
```

2. **Configure Readiness Check**:
```bash
gcloud run services update pixashot \
  --set-startup-check-path=/health/ready \
  --set-startup-initial-delay=15s \
  --set-startup-timeout=5s \
  --set-startup-failure-threshold=3
```

### Monitoring Alerts

```bash
# Create memory usage alert
gcloud beta monitoring alerts policies create \
  --display-name="Pixashot High Memory Usage" \
  --condition="resource.type = \"cloud_run_revision\" AND metric.type = \"run.googleapis.com/container/memory/utilization\" > 0.8" \
  --duration=300s

# Create error rate alert
gcloud beta monitoring alerts policies create \
  --display-name="Pixashot High Error Rate" \
  --condition="resource.type = \"cloud_run_revision\" AND metric.type = \"run.googleapis.com/request_count\" AND metric.labels.response_code_class = \"5xx\"" \
  --duration=60s

# Configure notification channels (optional)
gcloud beta monitoring channels create \
  --display-name="Email Notifications" \
  --type=email \
  --email-address=alerts@example.com
```

## Continuous Deployment

### Cloud Build Configuration

Create `cloudbuild.yaml`:
```yaml
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
      - '--set-secrets'
      - '/AUTH_TOKEN=pixashot-auth-token:latest'

substitutions:
  _SERVICE_NAME: pixashot
  _REGION: us-central1
  _MEMORY: 4Gi
  _CPU: "2"
  _TIMEOUT: "300"
```

### Automatic Deployments

Set up automatic deployments with triggers:
```bash
gcloud beta builds triggers create github \
  --repo-name=your-repo \
  --branch-pattern=^main$ \
  --build-config=cloudbuild.yaml
```

## Custom Domain Setup

1. **Verify Domain Ownership**:
```bash
gcloud domains verify example.com
```

2. **Map Custom Domain**:
```bash
gcloud beta run domain-mappings create \
  --service pixashot \
  --domain pixashot.example.com \
  --region us-central1
```

3. **Configure DNS**:
   Add the provided DNS records to your domain configuration.

## Cost Optimization

### Resource Configuration

Configure resources based on workload:

```bash
# Light workload (1-10 requests/minute)
--memory 2Gi --cpu 1 --concurrency 40 --min-instances 0 --max-instances 2

# Medium workload (10-50 requests/minute)
--memory 2Gi --cpu 2 --concurrency 80 --min-instances 1 --max-instances 5

# Heavy workload (50+ requests/minute)
--memory 4Gi --cpu 4 --concurrency 100 --min-instances 2 --max-instances 10
```

### Cost-Saving Tips

1. **Use Minimum Instances Wisely**:
```bash
# Development/Testing
--min-instances 0

# Production with predictable load
--min-instances 1
```

2. **Configure Concurrency**:
```bash
# Optimize for memory usage
--concurrency 80
```

3. **Enable Caching**:
```bash
--set-env-vars="CACHE_MAX_SIZE=1000"
```

## Troubleshooting

### Common Issues

1. **Cold Start Issues**:
- Increase minimum instances
- Reduce container size
- Optimize startup time

2. **Memory Issues**:
- Monitor memory metrics
- Adjust memory allocation
- Check for memory leaks

3. **Timeout Issues**:
- Adjust timeout settings
- Implement retry logic
- Check network dependencies

### Viewing Logs

```bash
# View recent logs
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=pixashot"

# Stream logs
gcloud logging tail "resource.type=cloud_run_revision AND resource.labels.service_name=pixashot"

# View error logs
gcloud logging read "resource.type=cloud_run_revision AND severity>=ERROR"
```

## Testing Deployment

```bash
# Get service URL
SERVICE_URL=$(gcloud run services describe pixashot --format='value(status.url)')

# Test health endpoints
curl $SERVICE_URL/health/live
curl $SERVICE_URL/health/ready

# Test screenshot capture
curl -X POST $SERVICE_URL/capture \
  -H "Authorization: Bearer your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "format": "png",
    "full_page": true
  }'
```

## Next Steps

- Set up [monitoring dashboards](https://console.cloud.google.com/monitoring/dashboards)
- Configure [logging exports](https://cloud.google.com/logging/docs/export)
- Implement [continuous deployment](https://cloud.google.com/build/docs/automating-builds/create-manual-triggers)
- Review [security best practices](../security/index.md)
- Set up [cost alerts](https://cloud.google.com/billing/docs/how-to/budgets)

For more information about specific features or configurations, refer to the [API Reference](../api-reference/index.md) and [Feature Documentation](../features/index.md).