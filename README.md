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
├── env_namespace.yaml          # Namespace definitions
├── external-deployment.yaml   # External API deployment
├── external-service.yaml      # External API service (LoadBalancer)
├── internal-deployment.yaml   # Internal API deployment
├── internal-service.yaml      # Internal API service (LoadBalancer)
└── README.md                  # This file
```

### File Descriptions

- **cluster.yaml**: Complete EKS cluster configuration with node groups and Fargate profiles
- **env_namespace.yaml**: Creates the 'development' namespace for application isolation
- **external-deployment.yaml**: Deploys the public-facing API service with 3 replicas
- **external-service.yaml**: LoadBalancer service exposing external API on port 80
- **internal-deployment.yaml**: Deploys the internal business logic service with 1 replica
- **internal-service.yaml**: LoadBalancer service exposing internal API on port 80
- **README.md**: Project documentation and deployment instructions

## Deployment

1. Create the EKS cluster:
   ```bash
   eksctl create cluster -f cluster.yaml
   ```

2. Create the development namespace:
   ```bash
   kubectl apply -f env_namespace.yaml
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

## How to Access Your Website

After successful deployment, you can access your services through the LoadBalancer DNS names:

### Get Service URLs
```bash
kubectl get services -n development
```

This will show output similar to:
```
NAME              TYPE           EXTERNAL-IP                                                              PORT(S)
events-external   LoadBalancer   aa11ea7edbc214c349a56ecce715b394-850688047.us-east-1.elb.amazonaws.com   80:30238/TCP
events-internal   LoadBalancer   ad9a763ae6ece4804b8b9c9c8c9a8b12-612744394.us-east-1.elb.amazonaws.com   80:31196/TCP
```

### Access Methods

**Web Browser:**
- Copy the `events-external` EXTERNAL-IP and paste it in your browser
- Example: `http://aa11ea7edbc214c349a56ecce715b394-850688047.us-east-1.elb.amazonaws.com`

### Troubleshooting Access Issues

If you cannot access the services:

1. **Check if pods are running:**
   ```bash
   kubectl get pods -n development
   ```

2. **Check application logs:**
   ```bash
   kubectl logs [POD-NAME] -n development
   ```

3. **Verify endpoints:**
   ```bash
   kubectl get endpoints -n development
   ```

**Note:** The applications must be listening on ports 8080 (external) and 8082 (internal) respectively for the services to work correctly.
