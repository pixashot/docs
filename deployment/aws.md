---
title: AWS Deployment
excerpt: Complete guide to deploying Pixashot on Amazon Web Services, including ECS/Fargate setup, monitoring, and AWS-specific optimizations.
meta:
  nav_order: 63
---

# AWS Deployment Guide

This guide covers deploying Pixashot on Amazon Web Services (AWS) using various container services and best practices.

## ECS vs Fargate Comparison

### AWS Fargate (Recommended)
- Serverless container management
- No infrastructure to maintain
- Automatic scaling
- Pay-per-use pricing
- Simplified security model

### Amazon ECS with EC2
- Full control over infrastructure
- Custom instance types
- Potentially lower costs at scale
- Advanced networking options
- Custom AMI support

## Fargate Deployment

### Prerequisites
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure
```

### Create Task Definition

```json
{
  "family": "pixashot",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "containerDefinitions": [
    {
      "name": "pixashot",
      "image": "gpriday/pixashot:latest",
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "AUTH_TOKEN",
          "value": "your_secret_token"
        },
        {
          "name": "WORKERS",
          "value": "4"
        },
        {
          "name": "MAX_REQUESTS",
          "value": "1000"
        }
      ],
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:8080/health/live || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/pixashot",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

## Load Balancer Configuration

### Application Load Balancer Setup

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name pixashot-alb \
  --subnets subnet-12345678 subnet-87654321 \
  --security-groups sg-12345678

# Create target group
aws elbv2 create-target-group \
  --name pixashot-tg \
  --protocol HTTP \
  --port 8080 \
  --vpc-id vpc-12345678 \
  --target-type ip \
  --health-check-path /health/live

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=$CERT_ARN \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN
```

### Health Check Configuration
```json
{
  "HealthCheckProtocol": "HTTP",
  "HealthCheckPort": "8080",
  "HealthCheckPath": "/health/live",
  "HealthCheckIntervalSeconds": 30,
  "HealthCheckTimeoutSeconds": 5,
  "HealthyThresholdCount": 2,
  "UnhealthyThresholdCount": 3
}
```

## Auto Scaling Configuration

### Service Auto Scaling

```bash
# Register scalable target
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/pixashot-cluster/pixashot \
  --min-capacity 1 \
  --max-capacity 10

# Configure scaling policy
aws application-autoscaling put-scaling-policy \
  --policy-name pixashot-cpu-scaling \
  --service-namespace ecs \
  --scalable-dimension ecs:service:DesiredCount \
  --resource-id service/pixashot-cluster/pixashot \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 70.0,
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
    }
  }'
```

## CloudWatch Monitoring

### Metric Collection

```bash
# Create log group
aws logs create-log-group --log-group-name /ecs/pixashot

# Create metric filter
aws logs put-metric-filter \
  --log-group-name /ecs/pixashot \
  --filter-name errors \
  --filter-pattern "ERROR" \
  --metric-transformations \
    metricName=ErrorCount,metricNamespace=Pixashot,metricValue=1
```

### CloudWatch Alarms

```bash
# CPU utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name pixashot-cpu-alarm \
  --alarm-description "CPU utilization exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions $SNS_TOPIC_ARN

# Memory utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name pixashot-memory-alarm \
  --alarm-description "Memory utilization exceeds 80%" \
  --metric-name MemoryUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions $SNS_TOPIC_ARN

# Error rate alarm
aws cloudwatch put-metric-alarm \
  --alarm-name pixashot-error-rate \
  --alarm-description "Error rate too high" \
  --metric-name ErrorCount \
  --namespace Pixashot \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions $SNS_TOPIC_ARN
```

### Dashboard Creation

```bash
aws cloudwatch put-dashboard \
  --dashboard-name "Pixashot-Dashboard" \
  --dashboard-body file://dashboard.json
```

Example `dashboard.json`:
```json
{
    "widgets": [
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/ECS", "CPUUtilization", "ServiceName", "pixashot"]
                ],
                "period": 300,
                "stat": "Average",
                "region": "us-west-2",
                "title": "CPU Utilization"
            }
        },
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/ECS", "MemoryUtilization", "ServiceName", "pixashot"]
                ],
                "period": 300,
                "stat": "Average",
                "region": "us-west-2",
                "title": "Memory Utilization"
            }
        }
    ]
}
```

## Cost Optimization

### Resource Optimization

1. **Right-sizing Containers**:
```json
{
  "cpu": "1024",
  "memory": "2048",
  "requiresCompatibilities": ["FARGATE"]
}
```

2. **Spot Instances** (for ECS EC2):
```bash
aws ecs create-capacity-provider \
  --name pixashot-spot \
  --auto-scaling-group-provider \
    "autoScalingGroupArn=$ASG_ARN,managedScaling={status=ENABLED,targetCapacity=100},managedTermination=true"
```

### Cost Monitoring

```bash
# Create budget
aws budgets create-budget \
  --account-id $ACCOUNT_ID \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

Example `budget.json`:
```json
{
    "BudgetName": "Pixashot-Monthly",
    "BudgetLimit": {
        "Amount": "100",
        "Unit": "USD"
    },
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST",
    "CostFilters": {
        "TagKeyValue": ["project:pixashot"]
    }
}
```

## Security Configuration

### IAM Roles

```bash
# Task execution role
aws iam create-role \
  --role-name pixashot-execution-role \
  --assume-role-policy-document file://task-execution-role.json

# Task role
aws iam create-role \
  --role-name pixashot-task-role \
  --assume-role-policy-document file://task-role.json
```

Example `task-execution-role.json`:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ecs-tasks.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

### Security Groups

```bash
# Create security group
aws ec2 create-security-group \
  --group-name pixashot-sg \
  --description "Security group for Pixashot containers"

# Configure inbound rules
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 8080 \
  --source $ALB_SG_ID
```

## Infrastructure as Code

### CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Pixashot ECS Fargate Deployment'

Parameters:
  Environment:
    Type: String
    Default: production
    AllowedValues: [development, staging, production]

Resources:
  PixashotCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: !Sub pixashot-${Environment}
      CapacityProviders: [FARGATE]

  PixashotTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: pixashot
      Cpu: 1024
      Memory: 2048
      NetworkMode: awsvpc
      RequiresCompatibilities: [FARGATE]
      ExecutionRoleArn: !Ref TaskExecutionRole
      ContainerDefinitions:
        - Name: pixashot
          Image: gpriday/pixashot:latest
          PortMappings:
            - ContainerPort: 8080
          Environment:
            - Name: AUTH_TOKEN
              Value: !Sub '{{resolve:secretsmanager:${PixashotSecret}:SecretString:auth_token}}'
          HealthCheck:
            Command: ["CMD-SHELL", "curl -f http://localhost:8080/health/live || exit 1"]
            Interval: 30
            Timeout: 5
            Retries: 3
            StartPeriod: 60

Outputs:
  ClusterArn:
    Description: 'ECS Cluster ARN'
    Value: !GetAtt PixashotCluster.Arn
```

### CDK Example

```typescript
import * as cdk from 'aws-cdk-lib';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as ecs_patterns from 'aws-cdk-lib/aws-ecs-patterns';

export class PixashotStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const cluster = new ecs.Cluster(this, 'PixashotCluster', {
      vpc: new ec2.Vpc(this, 'PixashotVPC')
    });

    const loadBalancedService = new ecs_patterns.ApplicationLoadBalancedFargateService(this, 'PixashotService', {
      cluster,
      memoryLimitMiB: 2048,
      cpu: 1024,
      taskImageOptions: {
        image: ecs.ContainerImage.fromRegistry('gpriday/pixashot:latest'),
        environment: {
          AUTH_TOKEN: process.env.AUTH_TOKEN || '',
          WORKERS: '4',
          MAX_REQUESTS: '1000'
        },
        containerPort: 8080
      }
    });
  }
}
```