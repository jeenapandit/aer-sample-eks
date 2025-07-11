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
- **Cluster Name**: `eks-jp`
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

- `jeenapandit/event-external:v1.0` - External API service
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

2. Create the development namespace:
   ```bash
   kubectl apply -f env_namespace.yml
   ```

3. Deploy the applications to the development namespace:
   ```bash
   kubectl apply -f internal-deployment.yaml -n development
   kubectl apply -f internal-service.yaml -n development
   kubectl apply -f external-deployment.yaml -n development
   kubectl apply -f external-service.yaml -n development
   ```

4. Verify the deployment:
   ```bash
   kubectl get pods -n development
   kubectl get services -n development
   ```

**Note:** The applications will be deployed to the `development` namespace. If you prefer to add the namespace directly to the YAML files, add `namespace: development` under the `metadata` section of each deployment and service file.