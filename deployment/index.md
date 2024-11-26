---
title: Deploying Pixashot
excerpt: Comprehensive guide to Pixashot deployment options, platform comparisons, and deployment strategy selection.
meta:
  nav_order: 150
---

# Deployment Overview

## Quick Platform Comparison

| Feature | Google Cloud Run | Docker Self-Hosted | AWS ECS/Fargate | Azure Container Apps |
|---------|-----------------|-------------------|------------------|---------------------|
| Management Overhead | Minimal | High | Medium | Medium |
| Auto-scaling | Yes | Manual | Yes | Yes |
| Cost Model | Pay-per-use | Fixed + Variable | Pay-per-use | Pay-per-use |
| Cold Starts | Yes | No | No | Yes |
| Max Memory | 32GB | Unlimited | 120GB | 32GB |
| Setup Complexity | Low | High | Medium | Low |
| Monitoring | Built-in | Manual | Built-in | Built-in |
| Offline Support | No | Yes | No | No |

## Platform Deep Dives

### Google Cloud Run (Recommended)
```bash
# Basic deployment
gcloud run deploy pixashot \
  --image gpriday/pixashot:latest \
  --memory 2Gi \
  --cpu 1 \
  --min-instances 1 \
  --max-instances 10
```

#### Ideal Use Cases
- Most production deployments
- Variable traffic patterns
- Cost-sensitive deployments
- Global distribution needs
- Serverless preferences

#### Key Benefits
- Zero infrastructure management
- Automatic scaling
- Pay-per-use pricing
- Built-in monitoring
- Global deployment options
- Easy integration with other GCP services

#### Limitations
- Cold starts (mitigatable)
- 32GB memory limit
- Regional deployment required
- No persistent storage

### Docker Self-Hosted
```bash
# Production deployment example
docker run -d \
  --name pixashot \
  --memory=4g \
  --cpus=2 \
  --restart=unless-stopped \
  -e AUTH_TOKEN=${AUTH_TOKEN} \
  -e WORKERS=4 \
  -p 8080:8080 \
  gpriday/pixashot:latest
```

#### Ideal Use Cases
- Local development
- Air-gapped environments
- Complete control requirements
- Existing infrastructure
- Specific compliance needs

#### Key Benefits
- Complete control
- No vendor lock-in
- Predictable costs
- Offline capability
- Custom hardware options
- Direct resource access

#### Limitations
- Manual scaling
- Infrastructure management
- Operational overhead
- Self-managed security
- Manual monitoring setup

### AWS ECS/Fargate
```bash
# Task definition example
{
  "family": "pixashot",
  "containerDefinitions": [{
    "name": "pixashot",
    "image": "gpriday/pixashot:latest",
    "memory": 2048,
    "cpu": 1024,
    "essential": true,
    "portMappings": [{
      "containerPort": 8080,
      "protocol": "tcp"
    }],
    "environment": [{
      "name": "AUTH_TOKEN",
      "value": "your_secret_token"
    }]
  }]
}
```

#### Ideal Use Cases
- AWS-integrated systems
- Complex networking needs
- Regional deployment requirements
- High availability needs
- Existing AWS expertise

#### Key Benefits
- AWS ecosystem integration
- Flexible scaling
- Rich monitoring
- VPC integration
- Route 53 integration
- Load balancer integration

#### Limitations
- Setup complexity
- Cost management needs
- AWS knowledge required
- Initial configuration time

### Azure Container Apps
```bash
# Basic deployment
az containerapp create \
  --name pixashot \
  --resource-group mygroup \
  --image gpriday/pixashot:latest \
  --target-port 8080 \
  --ingress external \
  --cpu 1.0 \
  --memory 2.0Gi
```

#### Ideal Use Cases
- Azure-integrated systems
- Simple deployment needs
- Microsoft ecosystem
- Existing Azure expertise

#### Key Benefits
- Azure ecosystem integration
- Simple deployment
- Built-in monitoring
- Azure DevOps integration
- Key Vault integration

#### Limitations
- Scale limitations
- Cost optimization needs
- Regional constraints
- Cold start impact

## Resource Planning

### Development Environment
```bash
Minimum Requirements:
- CPU: 1 core
- RAM: 2GB
- Storage: 1GB
- Network: Basic internet
```

### Production Environment
```bash
Recommended Requirements:
- CPU: 2+ cores
- RAM: 4GB+
- Storage: 5GB+
- Network: Low-latency
```

### High-Traffic Environment
```bash
Enterprise Requirements:
- CPU: 4+ cores
- RAM: 8GB+
- Storage: 10GB+
- Network: High bandwidth
```

## Cost Optimization Strategies

### Free Tier Maximization
1. **Google Cloud Run**
  - 2M requests/month free
  - Optimize instance count
  - Use minimum instances wisely
  - Monitor cold starts

2. **AWS Fargate**
  - Use Spot instances
  - Implement auto-scaling
  - Optimize task size
  - Use Application Load Balancer

3. **Azure Container Apps**
  - Use consumption plan
  - Optimize scaling rules
  - Monitor instance usage
  - Use reserved instances

### Cost Control Measures
```bash
# Example budget alert (GCP)
gcloud billing budgets create \
  --billing-account=$BILLING_ACCOUNT_ID \
  --display-name="Pixashot Budget" \
  --budget-amount=100 \
  --threshold-rule=percent=80
```

## Deployment Checklist

### Pre-Deployment
- [ ] Choose deployment platform
- [ ] Calculate resource requirements
- [ ] Plan monitoring strategy
- [ ] Set up security measures
- [ ] Configure backup strategy

### Deployment
- [ ] Set up authentication
- [ ] Configure networking
- [ ] Set resource limits
- [ ] Enable monitoring
- [ ] Configure scaling rules

### Post-Deployment
- [ ] Verify health checks
- [ ] Test performance
- [ ] Monitor resources
- [ ] Set up alerts
- [ ] Document deployment

## Common Issues & Solutions

### Performance Optimization
```bash
# Example monitoring setup (Docker)
docker stats pixashot --format \
  "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

#### Memory Issues
- Monitor usage patterns
- Set appropriate limits
- Implement cleanup routines
- Use proper scaling rules

#### Network Issues
- Configure timeouts
- Implement retry logic
- Use health checks
- Monitor latency

#### Security Concerns
- Implement authentication
- Configure network policies
- Enable rate limiting
- Regular security audits

## Monitoring Setup

### Health Endpoints
```bash
# Basic health monitoring
curl http://your-instance/health/live
curl http://your-instance/health/ready
curl http://your-instance/health/metrics
```

### Key Metrics
1. **System Health**
  - CPU utilization
  - Memory usage
  - Disk I/O
  - Network traffic

2. **Application Metrics**
  - Request latency
  - Error rates
  - Success rates
  - Queue length

3. **Business Metrics**
  - Capture success rate
  - Processing time
  - Cache hit ratio
  - Cost per capture

## Next Steps

### Platform-Specific Guides
- [Google Cloud Run Deployment](cloud-run.md)
- [Docker Self-Hosted Guide](docker.md)
- [AWS ECS/Fargate Setup](aws.md)
- [Azure Container Apps Guide](azure.md)

### Additional Resources
- [Scaling Strategies](scaling.md)
- [Security Configuration](../security/index.md)
- [Monitoring Setup](../deployment/cloud-run.md#monitoring)
- [Resource Management](../core-concepts/resource-management.md)