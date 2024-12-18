---
title: Azure Deployment Guide
excerpt: Comprehensive guide to deploying Pixashot on Microsoft Azure, including setup, configuration, monitoring, and production best practices.
meta:
  nav_order: 152
---

# Azure Deployment Guide

This guide provides comprehensive instructions for deploying Pixashot on Azure Container Apps, including initial setup, configuration, monitoring, and production best practices.

## Prerequisites

Before deploying Pixashot, ensure you have:
- Azure account
- Azure CLI installed
- Docker installed (optional for local testing)
- Resource group created
- Container Registry access

## Initial Setup

1. **Install Azure CLI**:
```bash
# For macOS
brew install azure-cli

# For Ubuntu/Debian
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# For Windows
# Download the MSI installer from Microsoft's website

# After installation
az login
az account set --subscription YOUR_SUBSCRIPTION_ID
```

2. **Enable Required Services**:
```bash
# Create resource group if not exists
az group create --name pixashot-rg --location eastus

# Create Container Registry
az acr create --resource-group pixashot-rg \
    --name pixashotregistry \
    --sku Basic \
    --admin-enabled true
```

## Basic Deployment

Deploy Pixashot with default settings using Azure Container Apps:

```bash
# Create Container App Environment
az containerapp env create \
    --name pixashot-env \
    --resource-group pixashot-rg \
    --location eastus

# Deploy Pixashot
az containerapp create \
    --name pixashot \
    --resource-group pixashot-rg \
    --environment pixashot-env \
    --image gpriday/pixashot:latest \
    --target-port 8080 \
    --ingress external \
    --cpu 1.0 \
    --memory 2.0Gi \
    --min-replicas 1 \
    --max-replicas 10 \
    --env-vars "AUTH_TOKEN=your_secret_token" \
    --registry-server docker.io
```

## Environment Configuration

### Required Variables
```bash
# Core Settings
AUTH_TOKEN=           # Authentication token
PORT=8080             # Server port

# Worker Configuration
WORKERS=4             # Number of worker processes
MAX_REQUESTS=1000     # Requests before worker restart
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
az containerapp create \
    --name pixashot \
    --resource-group pixashot-rg \
    --environment pixashot-env \
    --image gpriday/pixashot:latest \
    --target-port 8080 \
    --ingress external \
    --cpu 2.0 \
    --memory 4.0Gi \
    --min-replicas 2 \
    --max-replicas 10 \
    --replica-timeout 300 \
    --env-vars "AUTH_TOKEN=your_secret_token" \
                "USE_POPUP_BLOCKER=true" \
                "USE_COOKIE_BLOCKER=true" \
                "WORKERS=8" \
                "MAX_REQUESTS=1000" \
                "KEEP_ALIVE=300" \
                "RATE_LIMIT_ENABLED=true" \
                "RATE_LIMIT_CAPTURE=5 per second" \
                "CACHE_MAX_SIZE=1000"
```

## Security Configuration

### Secret Management

1. **Create Key Vault**:
```bash
# Create Key Vault
az keyvault create \
    --name pixashot-kv \
    --resource-group pixashot-rg \
    --location eastus

# Add secret
az keyvault secret set \
    --vault-name pixashot-kv \
    --name AUTH-TOKEN \
    --value "your_secret_token"
```

2. **Use Secrets in Deployment**:
```bash
# Get Key Vault ID
VAULT_ID=$(az keyvault show --name pixashot-kv \
    --query id --output tsv)

# Update Container App with secret
az containerapp update \
    --name pixashot \
    --resource-group pixashot-rg \
    --secret auth-token="secretref:$VAULT_ID/secrets/AUTH-TOKEN" \
    --set-env-vars "AUTH_TOKEN=secretref:auth-token"
```

### Role-Based Access Control (RBAC)

```bash
# Assign roles for Container App identity
az role assignment create \
    --assignee-object-id $IDENTITY_ID \
    --role "Key Vault Secrets User" \
    --scope $VAULT_ID
```

## Monitoring Setup

### Application Insights

1. **Create Application Insights**:
```bash
# Create Application Insights
az monitor app-insights component create \
    --app pixashot-insights \
    --location eastus \
    --resource-group pixashot-rg \
    --application-type web
```

2. **Configure Container App**:
```bash
# Get Instrumentation Key
INSTRUMENTATION_KEY=$(az monitor app-insights component show \
    --app pixashot-insights \
    --resource-group pixashot-rg \
    --query instrumentationKey \
    --output tsv)

# Update Container App
az containerapp update \
    --name pixashot \
    --resource-group pixashot-rg \
    --set-env-vars "APPLICATIONINSIGHTS_CONNECTION_STRING=$INSTRUMENTATION_KEY"
```

### Monitoring Alerts

```bash
# Create alert for high memory usage
az monitor metrics alert create \
    --name "High Memory Usage" \
    --resource-group pixashot-rg \
    --scopes $APP_ID \
    --condition "avg Memory > 80" \
    --window-size 5m \
    --evaluation-frequency 1m
```

## Continuous Deployment

### Azure DevOps Pipeline

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: pixashot-variables

steps:
  - task: Docker@2
    inputs:
      containerRegistry: 'pixashotregistry'
      repository: 'pixashot'
      command: 'build'
      Dockerfile: '**/Dockerfile'
      tags: |
        $(Build.BuildId)
        latest

  - task: AzureContainerApps@1
    inputs:
      azureSubscription: '$(AZURE_SUBSCRIPTION)'
      resourceGroup: 'pixashot-rg'
      containerAppName: 'pixashot'
      containerImage: 'pixashotregistry.azurecr.io/pixashot:$(Build.BuildId)'
      targetPort: 8080
```

## Cost Optimization

### Resource Configuration

Configure resources based on workload:

```bash
# Light workload (1-10 requests/minute)
--cpu 1.0 --memory 2.0Gi --min-replicas 0 --max-replicas 2

# Medium workload (10-50 requests/minute)
--cpu 2.0 --memory 2.0Gi --min-replicas 1 --max-replicas 5

# Heavy workload (50+ requests/minute)
--cpu 2.0 --memory 4.0Gi --min-replicas 2 --max-replicas 10
```

### Cost-Saving Tips

1. **Use Minimum Replicas Wisely**:
```bash
# Development/Testing
--min-replicas 0

# Production with predictable load
--min-replicas 1
```

2. **Configure Scale Rules**:
```bash
az containerapp scale rule add \
    --name pixashot \
    --resource-group pixashot-rg \
    --type http \
    --http-concurrency 50 \
    --min-replicas 0 \
    --max-replicas 10
```

## Troubleshooting

### Common Issues

1. **Cold Start Issues**:
- Increase minimum replicas
- Optimize container startup
- Review startup logs

2. **Memory Issues**:
- Monitor memory metrics
- Adjust memory allocation
- Check for memory leaks

3. **Scaling Issues**:
- Review scale rules
- Check resource quotas
- Monitor replica health

### Viewing Logs

```bash
# View application logs
az containerapp logs show \
    --name pixashot \
    --resource-group pixashot-rg \
    --follow

# View deployment logs
az containerapp logs deployment show \
    --name pixashot \
    --resource-group pixashot-rg
```

## Testing Deployment

```bash
# Get service URL
APP_URL=$(az containerapp show \
    --name pixashot \
    --resource-group pixashot-rg \
    --query properties.configuration.ingress.fqdn \
    --output tsv)

# Test health endpoints
curl https://$APP_URL/health/live
curl https://$APP_URL/health/ready

# Test screenshot capture
curl -X POST https://$APP_URL/capture \
    -H "Authorization: Bearer your_secret_token" \
    -H "Content-Type: application/json" \
    -d '{
        "url": "https://example.com",
        "format": "png",
        "full_page": true
    }'
```

## Next Steps

- Set up [monitoring dashboards](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/overview)
- Configure [diagnostic settings](https://docs.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)
- Implement [disaster recovery](https://docs.microsoft.com/en-us/azure/container-apps/disaster-recovery)
- Review [security best practices](../security/index.md)
- Set up [cost alerts](https://docs.microsoft.com/en-us/azure/cost-management-billing/costs/cost-mgt-alerts-monitor-usage-spending)

For more information about specific features or configurations, refer to the [API Reference](../api-reference/index.md) and [Feature Documentation](../api-reference/index.md).