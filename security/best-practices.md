---
title: Security Best Practices
excerpt: Comprehensive security guidelines and recommendations for deploying and maintaining Pixashot in production environments.
meta:
  nav_order: 113
---

# Security Best Practices

This guide provides essential security guidelines for deploying and maintaining Pixashot in production environments. These practices are designed to protect your screenshot service against common threats while ensuring reliable operation.

## Security Framework

### Authentication & Authorization

#### Token Management
```bash
# Generate cryptographically secure token
AUTH_TOKEN=$(openssl rand -base64 32)

# Store in cloud secret manager (Google Cloud example)
gcloud secrets create pixashot-auth-token \
  --data-file=- \
  --replication-policy="automatic"

# AWS Secrets Manager alternative
aws secretsmanager create-secret \
  --name pixashot-auth-token \
  --secret-string "$AUTH_TOKEN"

# Azure Key Vault alternative
az keyvault secret set \
  --vault-name pixashot-kv \
  --name AUTH-TOKEN \
  --value "$AUTH_TOKEN"
```

#### Token Best Practices
- Rotate tokens every 90 days
- Use different tokens per environment
- Implement token revocation procedures
- Monitor token usage patterns
- Store tokens in secure secret management services
- Use environment-specific token policies
- Implement token usage auditing

### Network Security

#### HTTPS Configuration
```nginx
# Nginx security headers
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Content-Security-Policy "default-src 'self'; img-src 'self' data:; script-src 'self';" always;
add_header Permissions-Policy "geolocation=(), camera=(), microphone=()" always;
```

#### Network Access Control
```yaml
# Cloud Run security configuration
steps:
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - 'pixashot'
      - '--ingress'
      - 'internal-and-cloud-load-balancing'
      - '--platform'
      - 'managed'
      - '--no-allow-unauthenticated'
      - '--vpc-connector'
      - 'projects/${PROJECT_ID}/locations/${REGION}/connectors/secure-connector'
```

#### Network Hardening Checklist
- [ ] Enable HTTPS-only access
- [ ] Configure secure response headers
- [ ] Implement request filtering
- [ ] Set up network isolation
- [ ] Configure proxy settings (if required)
- [ ] Enable DDoS protection
- [ ] Implement IP allowlisting
- [ ] Monitor network traffic patterns

### Container Security

#### Docker Security Configuration
```dockerfile
# Security-focused Dockerfile
FROM mcr.microsoft.com/playwright/python:v1.47.0

# Create non-root user
RUN useradd -r -s /bin/false -m -d /app appuser && \
    mkdir -p /app/data /app/logs && \
    chown -R appuser:appuser /app

# Set security-related environment variables
ENV NODE_OPTIONS="--no-experimental-fetch" \
    PYTHONPATH=/app \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

# Set resource limits
LIMIT_AS=1GB
LIMIT_CPU=1

# Switch to non-root user
USER appuser

WORKDIR /app
```

### Resource Protection

#### Memory Management
```bash
# Docker resource limits
docker run -d \
  --name pixashot \
  --memory="2g" \
  --memory-swap="2g" \
  --cpu-shares=1024 \
  --pids-limit=100 \
  --security-opt="no-new-privileges:true" \
  --cap-drop=ALL \
  gpriday/pixashot:latest
```

#### Resource Monitoring Configuration
```bash
# Create comprehensive monitoring policy
gcloud monitoring policies create \
  --display-name="Pixashot Resource Monitor" \
  --conditions='
    {
      "name": "memory-usage",
      "threshold": {
        "value": 85,
        "duration": "300s"
      },
      "filter": "metric.type=\"memory/utilization\""
    },
    {
      "name": "error-rate",
      "threshold": {
        "value": 5,
        "duration": "300s"
      },
      "filter": "metric.type=\"request_count\" AND metric.labels.status=\"5xx\""
    }
  '
```

### Comprehensive Security Monitoring

#### Core Metrics
```bash
# Health and security monitoring endpoints
curl http://your-instance/health | jq '.security'
curl http://your-instance/health/security | jq '.'
```

Monitor these key areas:
1. **Authentication**
    - Failed authentication attempts
    - Token usage patterns
    - Authentication bypasses
    - Session anomalies

2. **Network**
    - Request patterns
    - Response codes
    - Network latency
    - SSL/TLS errors
    - Proxy status

3. **Resources**
    - Memory utilization
    - CPU usage
    - Disk I/O
    - Network I/O
    - Process counts

4. **Security Events**
    - Failed captures
    - Access violations
    - Resource exhaustion
    - Configuration changes

### Incident Response Framework

#### 1. Preparation
- Document security procedures
- Configure monitoring alerts
- Establish response team
- Create communication channels
- Set up backup systems

#### 2. Detection
- Monitor security alerts
- Review system logs
- Track resource usage
- Analyze traffic patterns
- Check error rates

#### 3. Response
```bash
# Example incident response script
#!/bin/bash

# 1. Collect diagnostic information
docker logs pixashot > incident_$(date +%s).log
docker stats pixashot --no-stream > resources_$(date +%s).log

# 2. Implement temporary measures
docker update --memory=4g pixashot  # Increase resources
gcloud run services update-traffic pixashot --to-revisions=STABLE=100  # Rollback

# 3. Notify stakeholders
./notify-team.sh "Security incident detected - $(date)"
```

#### 4. Recovery
- Restore from backups
- Update configurations
- Rotate credentials
- Document lessons learned
- Update security measures

### Maintenance Schedule

#### Daily Tasks
- Monitor security alerts
- Review access logs
- Check resource usage
- Verify backup status

#### Weekly Tasks
- Review security events
- Analyze traffic patterns
- Update documentation
- Test monitoring systems

#### Monthly Tasks
- Rotate access tokens
- Update dependencies
- Review security policies
- Test recovery procedures

#### Quarterly Tasks
- Conduct security audit
- Update compliance documentation
- Review incident response plan
- Test backup restoration

### Security Compliance Checklist

#### Authentication & Access
- [ ] Implement token-based authentication
- [ ] Configure token rotation
- [ ] Set up access logging
- [ ] Enable rate limiting
- [ ] Implement IP restrictions

#### Network Security
- [ ] Enable HTTPS only
- [ ] Configure security headers
- [ ] Set up network isolation
- [ ] Implement request filtering
- [ ] Configure proxy settings

#### Resource Protection
- [ ] Set memory limits
- [ ] Configure CPU restrictions
- [ ] Enable resource monitoring
- [ ] Implement cleanup routines
- [ ] Configure backup processes

#### Monitoring & Response
- [ ] Set up security monitoring
- [ ] Configure alert policies
- [ ] Document incident procedures
- [ ] Enable audit logging
- [ ] Test recovery procedures

## Next Steps

1. **Platform-Specific Security**
    - [Google Cloud Run Security](../deployment/cloud-run.md#security-configuration)
    - [AWS Security](../deployment/aws.md#security-configuration)
    - [Azure Security](../deployment/azure.md#security-configuration)

2. **Additional Security Topics**
    - [Network Security Details](network-security.md)
    - [Authentication Configuration](authentication.md)
    - [Resource Management](../core-concepts/resource-management.md)

3. **Monitoring Setup**
    - [Monitoring Configuration](../deployment/cloud-run.md#monitoring)
    - [Alert Configuration](../deployment/cloud-run.md#monitoring)
    - [Log Management](../deployment/cloud-run.md#logging)