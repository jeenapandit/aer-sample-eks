# aer-sample-eks

## Project Overview

This project demonstrates a microservices architecture deployed on Amazon EKS (Elastic Kubernetes Service) using a hybrid compute model with both managed node groups and Fargate profiles.

## Architecture

The project consists of two main microservices:

- **External API Service** (`events-external`): Public-facing API that serves as the entry point
- **Internal API Service** (`events-internal`): Internal service handling business logic

Both services communicate with each other, with the external service making calls to the internal service.

## Infrastructure Components

### EKS Cluster Configuration
- **Cluster Name**: `jp`
- **Region**: `us-east-1`
- **Kubernetes Version**: `1.30`
- **Availability Zones**: `us-east-1a`, `us-east-1b`

### Compute Resources
- **Managed Node Group**: Spot instances (`t2.medium`) with auto-scaling (2-4 nodes)
- **Fargate Profiles**: Serverless compute for specific namespaces

### Add-ons
- VPC CNI
- CoreDNS
- Kube Proxy
- AWS EBS CSI Driver
- Amazon CloudWatch Observability

### IAM Integration
- OIDC provider enabled
- Service accounts for cluster autoscaler and AWS Load Balancer Controller

## Docker Images

- `jeenapandit/external-api:v1.0` - External API service
- `jeenapandit/event-internal:v1.0` - Internal API service

## File Structure

```
├── cluster.yaml              # EKS cluster configuration
├── env_namespace.yml          # Namespace definitions
├── external-deployment.yaml   # External API deployment
├── external-service.yaml      # External API service (LoadBalancer)
├── internal-deployment.yaml   # Internal API deployment
├── internal-service.yaml      # Internal API service (LoadBalancer)
└── README.md                  # This file
```

## Deployment

1. Create the EKS cluster:
   ```bash
   eksctl create cluster -f cluster.yaml
   ```

2. Deploy the applications:
   ```bash
   kubectl apply -f env_namespace.yml
   kubectl apply -f internal-deployment.yaml
   kubectl apply -f internal-service.yaml
   kubectl apply -f external-deployment.yaml
   kubectl apply -f external-service.yaml
   ```

## Monitoring

CloudWatch logging is enabled for the following components:
- API server
- Audit logs
- Authenticator
- Controller Manager
- Scheduler