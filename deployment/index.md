---
title: Deploying Pixashot
excerpt: Overview of Pixashot deployment options, platform comparisons, and choosing the right deployment strategy for your needs.
meta:
    nav_order: 150
---

# Deployment Overview

Pixashot offers flexible deployment options to suit different scales and requirements. This guide helps you choose the right deployment strategy and understand the implications of each option.

## Deployment Options

### Google Cloud Run (Recommended)
- **Best for**: Most users, from small to large scale
- **Advantages**:
    - Zero infrastructure management
    - Automatic scaling
    - Pay-per-use pricing
    - Built-in monitoring
    - Global deployment options
- **Considerations**:
    - Cold starts on free tier
    - Regional availability
    - Memory limits (up to 32GB)

### Docker
- **Best for**: Local development and self-hosted environments
- **Advantages**:
    - Complete control
    - No vendor lock-in
    - Local development ease
    - Predictable costs
- **Considerations**:
    - Manual scaling
    - Infrastructure management required
    - More operational overhead

### AWS ECS/Fargate
- **Best for**: AWS-integrated environments
- **Advantages**:
    - AWS ecosystem integration
    - Flexible scaling options
    - Regional availability
    - Rich monitoring tools
- **Considerations**:
    - Higher complexity
    - Cost management needed
    - Infrastructure setup required

### Azure Container Instances
- **Best for**: Azure-integrated environments
- **Advantages**:
    - Azure ecosystem integration
    - Simple deployment
    - Good monitoring tools
    - Regional availability
- **Considerations**:
    - Limited scaling options
    - Cost optimization needed
    - Resource limits

## Resource Requirements

### Minimum Requirements
- CPU: 1 core
- RAM: 2GB
- Storage: 1GB
- Network: Reliable internet connection

### Recommended Production Requirements
- CPU: 2+ cores
- RAM: 4GB+
- Storage: 5GB+
- Network: Low-latency connection

### Scaling Factors
- Number of concurrent users
- Screenshot frequency
- Page complexity
- Output format requirements
- Caching needs

## Cost Considerations

### Google Cloud Run
- Free tier includes:
    - 2 million requests/month
    - 360,000 vCPU-seconds
    - 180,000 GiB-seconds
- Beyond free tier:
    - $0.00002400/vCPU-second
    - $0.00000250/GiB-second
    - $0.40/million requests

### Self-Hosted Docker
- Server costs
- Maintenance overhead
- Monitoring costs
- Backup costs
- Network transfer

### AWS
- Container service costs
- Load balancer costs
- Network transfer
- Monitoring costs
- Storage costs

### Azure
- Container instance costs
- Network transfer
- Monitoring costs
- Storage costs

## Platform Selection Guide

### Choose Google Cloud Run if:
- You want minimal management overhead
- Your usage patterns are irregular
- You need automatic scaling
- Cost optimization is important
- You want easy deployment

### Choose Docker Self-Hosted if:
- You need complete control
- You have existing infrastructure
- You have specific compliance requirements
- You want predictable costs
- You need offline capabilities

### Choose AWS if:
- You're heavily invested in AWS
- You need AWS-specific features
- You want regional deployment
- You need complex auto-scaling
- You use other AWS services

### Choose Azure if:
- You're heavily invested in Azure
- You need Azure-specific features
- You want simple container deployment
- You use other Azure services

## Common Pitfalls and Solutions

### Resource Allocation
**Pitfall**: Underestimating memory requirements
**Solution**:
- Start with at least 2GB RAM
- Monitor memory usage
- Set appropriate limits
- Use scaling rules

### Cold Starts
**Pitfall**: Slow initial responses
**Solution**:
- Configure minimum instances
- Use warm-up endpoints
- Implement caching
- Optimize startup time

### Cost Management
**Pitfall**: Unexpected costs
**Solution**:
- Set up budget alerts
- Monitor usage patterns
- Implement auto-scaling limits
- Use cost optimization tools

### Network Issues
**Pitfall**: Timeouts and failures
**Solution**:
- Configure appropriate timeouts
- Implement retry logic
- Use health checks
- Monitor network metrics

### Security Configuration
**Pitfall**: Exposed services
**Solution**:
- Use authentication tokens
- Configure network policies
- Implement rate limiting
- Regular security audits

## Health Monitoring

Every deployment should include:

1. **Basic Health Checks**
```bash
curl http://your-instance/health/live
curl http://your-instance/health/ready
```

2. **Resource Monitoring**
- CPU usage
- Memory usage
- Network traffic
- Request latency

3. **Alert Configuration**
- Error rate thresholds
- Resource utilization
- Response time
- System availability

## Next Steps

Based on your deployment choice:
- [Google Cloud Run Deployment Guide](cloud-run.md)
- [Docker Deployment Guide](docker.md)
- [AWS Deployment Guide](aws.md)
- [Azure Deployment Guide](azure.md)
- [Scaling Guide](scaling.md)

Each platform has its own considerations and best practices. Choose the one that best fits your needs and follow the respective deployment guide.