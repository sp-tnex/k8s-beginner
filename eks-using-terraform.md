# Kubernetes Local + AWS EKS Setup Using Terraform

## Overview

This guide explains how to:

- Create AWS infrastructure using Terraform
- Provision an EKS cluster with Terraform
- Configure kubectl access
- Create local Kubernetes clusters for development
- Push Docker images to ECR
- Deploy applications to EKS
- Maintain Infrastructure as Code (IaC)

This is the industry-standard approach.

## Why Terraform?

Terraform solves a major problem:

Without Terraform:

- manual AWS clicking
- inconsistent infrastructure
- hard-to-reproduce environments
- impossible team scaling

With Terraform:

- infrastructure becomes version-controlled
- reproducible environments
- automation-ready
- safer deployments
- easier rollback

## Install Required Tools

### 1. Install Terraform

Official Website:

[Terraform by HashiCorp](https://developer.hashicorp.com/terraform)

Verify:

```bash
terraform version
```

### 2. Install Docker

[Docker Desktop](https://www.docker.com/products/docker-desktop/?utm_source=chatgpt.com)

Verify:

```bash
docker --version
```

### 3. Install kubectl

[kubectl Installation](https://kubernetes.io/docs/tasks/tools/?utm_source=chatgpt.com)

Verify:

```bash
kubectl version --client
```

### 4. Install AWS CLI

[AWS CLI](https://aws.amazon.com/cli/?utm_source=chatgpt.com)

Verify:

```bash
aws --version
```

### 5. Install kind

[kind Official Site](https://kind.sigs.k8s.io/?utm_source=chatgpt.com)

Verify:

```bash
kind version
```

## Configure AWS Credentials

Run:

```bash
aws configure
```

Provide:

- AWS Access Key
- AWS Secret Key
- Region
- Output format

Example:

```text
Region: ap-south-1
Output: json
```

## Initialize Terraform

```bash
terraform init
```

## Validate Configuration

```bash
terraform validate
```

## Preview Infrastructure

```bash
terraform plan
```

Read the plan carefully.

Never blindly apply Terraform.

## Create Infrastructure

```bash
terraform apply
```

Type:

```text
yes
```

Provisioning takes:

- 15–25 minutes

## Configure kubectl Access

After cluster creation:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name production-eks
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
NAME                          STATUS   ROLES    AGE
ip-10-0-x-x.ec2.internal     Ready    <none>   5m
```

## Create Local Kubernetes Cluster

Local development still matters.

Do NOT use AWS for every test.

### Create kind Cluster

```bash
kind create cluster --name dev-cluster
```

Verify:

```bash
kubectl cluster-info --context kind-dev-cluster
```

## Dockerize Application

### Dockerfile

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

### Load Image into kind

```bash
kind load docker-image k8s-demo:v1 --name dev-cluster
```

## Kubernetes Deployment

### Deploy to Local Cluster

```bash
kubectl apply -f deployment.yaml
```

### Access Application

```bash
kubectl port-forward deployment/k8s-demo 3000:3000
```

Open:

```text
http://localhost:3000
```

## Create AWS ECR Repository

### Terraform Option

Add to main.tf:

```hcl
resource "aws_ecr_repository" "myapp" {
  name = "myapp"
}
```

Apply again:

```bash
terraform apply
```

## Authenticate Docker to ECR

```bash
aws ecr get-login-password \
| docker login \
  --username AWS \
  --password-stdin YOUR_ECR_URL
```

## Push Docker Image

### Tag Image

```bash
docker tag myapp:v1 \
YOUR_ECR_URL/myapp:v1
```

### Push Image

```bash
docker push YOUR_ECR_URL/myapp:v1
```

## Deploy to AWS EKS

### Update deployment.yaml

Replace:

```yaml
image: myapp:v1
```

With:

```yaml
image: YOUR_ECR_URL/myapp:v1
```

Remove:

```yaml
imagePullPolicy: Never
```

### Change Service Type

```yaml
type: LoadBalancer
```

### Deploy

```bash
kubectl apply -f kubernetes/deployment.yaml
```

## Get Public Endpoint

```bash
kubectl get svc
```

Look for:

```text
EXTERNAL-IP
```

Open in browser.

## Switch Between Local and AWS Clusters

This is extremely important.

### View Contexts

```bash
kubectl config get-contexts
```

### Switch to Local

```bash
kubectl config use-context kind-dev-cluster
```

### Switch to AWS

```bash
kubectl config use-context arn:aws:eks:ap-south-1:ACCOUNT_ID:cluster/production-eks
```

## Terraform Workflow

### Typical Workflow

```text
Edit Terraform
    ↓
terraform plan
    ↓
Review changes
    ↓
terraform apply
```

## Destroy Infrastructure

Critical for cost control.

### Delete Everything

```bash
terraform destroy
```

This deletes:

- EKS cluster
- VPC
- Node groups
- networking
- ECR resources

Always clean up unused infrastructure.
