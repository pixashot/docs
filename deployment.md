---
excerpt: "Comprehensive overview of deployment options for Pixashot, including cloud platforms, self-hosted solutions, and scaling considerations."
published_at: true
---

# Deployment

Pixashot can be deployed on various cloud platforms or self-hosted environments. This section provides an overview of deployment options and links to detailed documentation for each platform.

## Google Cloud Run

Google Cloud Run is an excellent option for deploying Pixashot, offering easy scaling and a generous free tier.

- [Deploying container images on Cloud Run](https://cloud.google.com/run/docs/deploying)
- [Setting up continuous deployment](https://cloud.google.com/run/docs/continuous-deployment)
- [Managing secrets in Cloud Run](https://cloud.google.com/run/docs/configuring/secrets)

Note: A dedicated guide for deploying Pixashot on Google Cloud Run will be available soon.

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
docker pull gpriday/pixashot:latest
docker run -p 8080:8080 -e AUTH_TOKEN=your_secret_token gpriday/pixashot:latest
```

For more details on Docker deployment, refer to:
- [Docker documentation](https://docs.docker.com/)
- [Docker Compose documentation](https://docs.docker.com/compose/)

## Scaling Considerations

When deploying Pixashot, consider the following for optimal performance and scalability:

1. **Resource Allocation**: Ensure adequate CPU and memory for each instance.
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