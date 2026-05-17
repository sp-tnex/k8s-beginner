# Kubernetes Beginner Project

A simple Node.js + Kubernetes project to learn containerization, deployments, services, scaling, and debugging using Minikube or EC2.

# What You Will Learn

This project teaches practical Kubernetes concepts by deploying a real application.

You will learn:

- Docker basics
- Containerization
- Kubernetes Deployments
- Services
- Scaling
- Rolling updates
- Debugging pods
- Kubernetes networking
- Local cluster setup with Minikube
- EC2 deployment basics

# Prerequisites

Install these tools before starting.

## Required Tools

- Docker
- kubectl
- Minikube
- Node.js

Official Documentation:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/?utm_source=chatgpt.com)
- [kubectl Install Guide](https://kubernetes.io/docs/tasks/tools/?utm_source=chatgpt.com)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/?utm_source=chatgpt.com)
- [Node.js](https://nodejs.org/?utm_source=chatgpt.com)

# Step 1: Start Kubernetes Cluster

Start Minikube:

```bash
minikube start
```

Check cluster status:

```bash
kubectl get nodes
```

## What This Means

### `minikube start`

Creates a local Kubernetes cluster on your machine.

### `kubectl get nodes`

Shows available Kubernetes nodes.

Expected output:

```bash
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...
```

# Step 2: Install Dependencies

```bash
npm install
```

## Why This Is Needed

Installs Node.js dependencies from `package.json`.

Without this:

- Express server will not run
- Application cannot start

# Step 3: Run Application Locally

```bash
node server.js
```

Open:

```text
http://localhost:3000
```

## What This Does

Starts the Node.js Express server locally without Docker or Kubernetes.

This helps verify:

- App works correctly
- No syntax/runtime errors

# Step 4: Build Docker Image

```bash
docker build -t k8s-demo:v1 .
```

## What This Means

### `docker build`

Creates a Docker image from the Dockerfile.

### `-t k8s-demo:v1`

Tags the image:

- `k8s-demo` = image name
- `v1` = version tag

### `.`

Uses current directory as build context.

# Step 5: Run Docker Container

```bash
docker run -p 3000:3000 k8s-demo:v1
```

## Why This Is Important

Tests container before Kubernetes deployment.

### `-p 3000:3000`

Maps:

- local machine port → container port

Format:

```text
host-port:container-port
```

# Step 6: Use Minikube Docker Environment

```bash
eval $(minikube docker-env)
```

## Why This Is Required

Minikube uses its own internal Docker daemon.

Without this:

- Kubernetes cannot find local images
- Pods fail with:

```text
ErrImageNeverPull
```

This command switches your terminal to Minikube's Docker environment.

# Step 7: Build Image Inside Minikube

```bash
docker build -t k8s-demo:v1 .
```

## Why Build Again?

Now image is created inside Minikube.

Kubernetes pods can access it successfully.

# Step 8: Deploy Application

```bash
kubectl apply -f deployment.yaml
```

## What Is deployment.yaml?

Defines:

- number of replicas
- container image
- ports
- pod labels

## Why Use Deployments?

Deployments provide:

- automatic recovery
- scaling
- rolling updates
- self-healing

# Step 9: Check Pods

```bash
kubectl get pods
```

## What Are Pods?

Pods are the smallest deployable unit in Kubernetes.

Each pod contains:

- one or more containers

Expected output:

```bash
NAME                         READY   STATUS
k8s-demo-xxxxx              1/1     Running
```

# Step 10: Create Service

```bash
kubectl apply -f service.yaml
```

## What Is a Service?

Pods are temporary.

Services provide:

- stable networking
- load balancing
- access to pods

Without services:

- pod IP changes break communication

# Step 11: Access Application

```bash
minikube service k8s-demo-service
```

## What This Does

Exposes Kubernetes service in browser.

Minikube automatically:

- creates URL
- forwards traffic

# Kubernetes Commands Explained

## Get Pods

```bash
kubectl get pods
```

Shows running pods.

## Describe Pod

```bash
kubectl describe pod POD_NAME
```

Shows:

- events
- errors
- image issues
- scheduling details

Most useful debugging command.

## View Logs

```bash
kubectl logs POD_NAME
```

Shows application logs inside container.

Useful for:

- crashes
- errors
- debugging

## Execute Inside Container

```bash
kubectl exec -it POD_NAME -- sh
```

Opens shell inside container.

Useful for:

- checking files
- testing connectivity
- debugging runtime issues

## Scale Deployment

```bash
kubectl scale deployment k8s-demo --replicas=5
```

Increases running pod count.

Kubernetes automatically creates additional pods.

## Delete Pod

```bash
kubectl delete pod POD_NAME
```

Kubernetes automatically recreates deleted pod.

This demonstrates:

- self-healing
- reconciliation loop

## Check All Resources

```bash
kubectl get all
```

Shows:

- pods
- services
- deployments
- replica sets

Useful for quick cluster overview.

# Common Errors

## ErrImageNeverPull

### Cause

Kubernetes cannot find Docker image.

### Fix

```bash
eval $(minikube docker-env)
docker build -t k8s-demo:v1 .
```

## ImagePullBackOff

### Cause

Wrong image name or missing Docker registry image.

### Fix

- verify image exists
- check Docker Hub image
- check image tag

## CrashLoopBackOff

### Cause

Application crashes repeatedly.

### Debug

```bash
kubectl logs POD_NAME
```

# Dockerfile Explanation

```dockerfile
FROM node:18-alpine
```

Uses lightweight Node.js base image.

```dockerfile
WORKDIR /app
```

Sets working directory inside container.

```dockerfile
COPY package*.json ./
```

Copies dependency files.

```dockerfile
RUN npm install
```

Installs dependencies.

```dockerfile
COPY . .
```

Copies application code.

```dockerfile
EXPOSE 3000
```

Documents container port.

```dockerfile
CMD ["npm", "start"]
```

Starts application.

# deployment.yaml Explanation

```yaml
replicas: 2
```

Runs 2 pod instances.

```yaml
selector:
  matchLabels:
    app: k8s-demo
```

Deployment finds pods using labels.

```yaml
labels:
  app: k8s-demo
```

Labels identify pods.

```yaml
image: k8s-demo:v1
```

Container image to run.

```yaml
containerPort: 3000
```

Application listens on port 3000.

# service.yaml Explanation

```yaml
type: NodePort
```

Exposes application externally.

```yaml
port: 80
```

Service port.

```yaml
targetPort: 3000
```

Forwards traffic to container port 3000.

# Real Kubernetes Concepts Learned

This project demonstrates:

| Concept      | Meaning                        |
| ------------ | ------------------------------ |
| Pod          | Running container instance     |
| Deployment   | Manages pod lifecycle          |
| Service      | Stable networking layer        |
| ReplicaSet   | Maintains desired pod count    |
| Labels       | Resource identification        |
| Selectors    | Connect resources together     |
| Scaling      | Increase/decrease pods         |
| Self-healing | Auto recreation of failed pods |

# Run on EC2

Install lightweight Kubernetes:

```bash
curl -sfL https://get.k3s.io | sh -
```

Check cluster:

```bash
sudo kubectl get nodes
```

# Push Docker Image to Docker Hub

Tag image:

```bash
docker tag k8s-demo:v1 YOUR_USERNAME/k8s-demo:v1
```

Push image:

```bash
docker push YOUR_USERNAME/k8s-demo:v1
```

Update deployment image:

```yaml
image: YOUR_USERNAME/k8s-demo:v1
```

# Useful Resources

- [Kubernetes Official Docs](https://kubernetes.io/docs/home/?utm_source=chatgpt.com)
- [Docker Documentation](https://docs.docker.com/?utm_source=chatgpt.com)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/?utm_source=chatgpt.com)
