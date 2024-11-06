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
  --set-env-vars="CLOUD_RUN=true,AUTH_TOKEN=your_secret_token" \
  --memory 2Gi \
  --cpu 2
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

When deploying to AWS, remember to set `CLOUD_RUN=false` in your environment variables.

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
  -e CLOUD_RUN=false \
  gpriday/pixashot:latest

# Advanced deployment with resource limits
docker run -p 8080:8080 \
  -e AUTH_TOKEN=your_secret_token \
  -e CLOUD_RUN=false \
  -e WORKERS=4 \
  -e MEMORY_LIMIT=2048 \
  -e TIMEOUT=300 \
  --memory=2g \
  gpriday/pixashot:latest
```

For more details on Docker deployment, refer to:
- [Docker documentation](https://docs.docker.com/)
- [Docker Compose documentation](https://docs.docker.com/compose/)

## Environment Variables

Key environment variables for configuring your deployment:

- `CLOUD_RUN`: Set to "true" for Google Cloud Run deployments, "false" otherwise
- `WORKERS`: Number of worker processes (default: 4)
- `MEMORY_LIMIT`: Memory limit in MB (default: 2048)
- `TIMEOUT`: Request timeout in seconds (default: 300)
- `AUTH_TOKEN`: Your authentication token
- `PORT`: Server port (default: 8080)

## Scaling Considerations

When deploying Pixashot, consider the following for optimal performance and scalability:

1. **Resource Allocation**:
    - Ensure adequate CPU (minimum 1 core recommended)
    - Allocate sufficient memory (minimum 2GB per instance)
    - Consider your concurrent request requirements when sizing instances

2. **Concurrency**: Adjust the number of workers based on your expected load.
3. **Statelessness**: Design your deployment to be stateless for easy horizontal scaling.
4. **Load Balancing**: Implement a load balancer for distributing traffic across multiple instances.
5. **Monitoring**: Set up monitoring and alerting to track performance and usage.

For platform-specific scaling best practices, refer to:
- [Google Cloud Run Scalability](https://cloud.google.com/run/docs/about-instance-autoscaling)
- [AWS ECS Scalability](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)
- [Azure Container Instances Scalability](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-orchestrator-relationship)
- [Kubernetes Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

Remember to always test your deployment thoroughly in a staging environment before going live. Each platform has its own set of best practices and optimization techniques, so be sure to consult the specific documentation for your chosen deployment method.

## Health Checks

The service provides two health check endpoints:
- `/health/live`: Basic liveness check
- `/health/ready`: Readiness check including browser availability

Configure these in your platform's health check settings to ensure proper monitoring of your deployment.