# Kubernetes Local + AWS EKS Setup Guide

## Overview

This guide explains how to:

- Create a local Kubernetes cluster
- Run applications locally
- Create an AWS EKS cluster
- Push Docker images to AWS ECR
- Deploy the same application to AWS
- Keep local and cloud environments aligned

This is designed as a practical beginner-to-intermediate workflow.

## Architecture

```text
Local Machine
│
├── Docker
├── kind (Local Kubernetes)
├── kubectl
├── Application Code
│
└── Push to GitHub + AWS ECR
          │
          ▼
AWS EKS Cluster
```

## Tech Stack

| Tool    | Purpose                  |
| ------- | ------------------------ |
| Docker  | Containerization         |
| kind    | Local Kubernetes cluster |
| kubectl | Kubernetes CLI           |
| AWS EKS | Managed Kubernetes       |
| AWS ECR | Container registry       |
| eksctl  | EKS cluster management   |
| GitHub  | Source control           |

## Install Required Tools

### 1. Install Docker

Official Website:

[Docker Desktop](https://www.docker.com/products/docker-desktop/)

Verify:

```bash
docker --version
```

### 2. Install kubectl

Official Guide:

[kubectl Installation](https://kubernetes.io/docs/tasks/tools/)

Verify:

```bash
kubectl version --client
```

### 3. Install kind

Official Website:

[kind Official Site](https://kind.sigs.k8s.io/)

Verify:

```bash
kind version
```

### 4. Install AWS CLI

Official Website:

[AWS CLI](https://aws.amazon.com/cli/)

Verify:

```bash
aws --version
```

### 5. Install eksctl

Official Website:

[eksctl Official Site](https://eksctl.io/)

Verify:

```bash
eksctl version
```

## Configure AWS

Run:

```bash
aws configure
```

Enter:

- AWS Access Key
- AWS Secret Key
- Region
- Output format

Example:

```text
Region: ap-south-1
Output: json
```

## Create Local Kubernetes Cluster

### Create Cluster

```bash
kind create cluster --name dev-cluster
```

### Verify Cluster

```bash
kubectl cluster-info
kubectl get nodes
```

Expected:

```text
NAME                         STATUS   ROLES           AGE
dev-cluster-control-plane   Ready    control-plane   1m
```

## Dockerize Application

### Create Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm","start"]
```

### Build Docker Image

```bash
docker build -t k8s-demo:v1 .
```

Verify:

```bash
docker images
```

### Load Docker Image into kind

Important:

kind cannot automatically access local Docker images.

Load manually:

```bash
kind load docker-image k8s-demo:v1 --name dev-cluster
```

## Apply Configuration

```bash
kubectl apply -f deployment.yaml
```

## Verify Deployment

```bash
kubectl get pods
kubectl get services
```

## Access Application Locally

### Port Forward

```bash
kubectl port-forward deployment/k8s-demo 3000:3000
```

Open:

```text
http://localhost:3000
```

Expected Output:

```text
Hello Kubernetes
```

## Create AWS EKS Cluster

### Warning

EKS costs money.

Approximate beginner setup cost:

- $60–120/month if left running

Always destroy unused clusters.

### Create Cluster : eksctl

```bash
eksctl create cluster \
  --name production \
  --region ap-south-1 \
  --nodes 2
```

This takes 15–20 minutes.

### Verify Cluster : eksctl

```bash
kubectl get nodes
```

You should see AWS EC2 worker nodes.

## Create AWS ECR Repository

### Create Repository

```bash
aws ecr create-repository \
  --repository-name myapp
```

### Authenticate Docker to ECR

```bash
aws ecr get-login-password \
| docker login \
  --username AWS \
  --password-stdin YOUR_ECR_URL
```

Example:

```text
123456789.dkr.ecr.ap-south-1.amazonaws.com
```

### Push Docker Image to ECR

### Tag Image

```bash
docker tag myapp:v1 \
YOUR_ECR_URL/myapp:v1
```

### Push Image

```bash
docker push YOUR_ECR_URL/myapp:v1
```

## Deploy Application to AWS

### Update deployment.yaml

Replace:

```yaml
image: myapp:v1
```

With:

```yaml
image: YOUR_ECR_URL/myapp:v1
```

Also remove:

```yaml
imagePullPolicy: Never
```

because AWS nodes will pull from ECR.

### Deploy

```bash
kubectl apply -f deployment.yaml
```

## Expose Application Publicly

### Change Service Type

Replace:

```yaml
type: ClusterIP
```

With:

```yaml
type: LoadBalancer
```

Apply again:

```bash
kubectl apply -f deployment.yaml
```

### Get Public URL

```bash
kubectl get svc
```

Look for:

```text
EXTERNAL-IP
```

Open that IP in browser.

## Common kubectl Commands

### Get Pods

```bash
kubectl get pods
```

### Describe Pod

```bash
kubectl describe pod POD_NAME
```

### View Logs

```bash
kubectl logs POD_NAME
```

### Restart Deployment

```bash
kubectl rollout restart deployment myapp
```

### Delete Deployment

```bash
kubectl delete deployment myapp
```

## Recommended Development Workflow

### Local Development

```text
Code Change
    ↓
Docker Build
    ↓
Load into kind
    ↓
kubectl apply
    ↓
Test locally
```

## Production Workflow

```text
Code Change
    ↓
Git Push
    ↓
CI/CD Pipeline
    ↓
Build Docker Image
    ↓
Push to ECR
    ↓
Deploy to EKS
```

## CI/CD Next Steps

Recommended tools later:

| Tool           | Purpose                |
| -------------- | ---------------------- |
| GitHub Actions | CI/CD                  |
| Helm           | Kubernetes templating  |
| ArgoCD         | GitOps deployments     |
| Terraform      | Infrastructure as code |

## Important Kubernetes Concepts

| Concept    | Meaning                  |
| ---------- | ------------------------ |
| Pod        | Smallest deployable unit |
| Deployment | Manages pods             |
| Service    | Network access           |
| Ingress    | HTTP routing             |
| Namespace  | Logical isolation        |
| ConfigMap  | Non-secret configs       |
| Secret     | Sensitive data           |
| Node       | Worker machine           |
| Cluster    | Group of nodes           |

## Delete AWS Cluster

Very important.

When finished:

```bash
eksctl delete cluster \
  --name production \
  --region ap-south-1
```

## Useful Resources

### Kubernetes Documentation

[Kubernetes Docs](https://kubernetes.io/docs/)

### AWS EKS Documentation

[AWS EKS Docs](https://docs.aws.amazon.com/eks/)

### Docker Documentation

[Docker Docs](https://docs.docker.com/)
