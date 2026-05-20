# Kubernetes Beginner Project using kind

A simple Node.js + Kubernetes project to learn containerization, deployments, services, scaling, and debugging using kind.

# What is kind?

kind stands for:

```text
Kubernetes IN Docker
```

kind runs Kubernetes clusters using Docker containers as nodes.

It is:

- lightweight
- fast
- beginner friendly
- commonly used for CI/CD testing
- useful for local Kubernetes practice

# Prerequisites

Install these tools before starting.

## Required Tools

- Docker
- kubectl
- kind
- Node.js

Official Documentation:

- [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)
- [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)
- [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
- [https://nodejs.org/](https://nodejs.org/)

# Step 1: Create Kubernetes Cluster

Create cluster:

```bash
kind create cluster --name k8s-demo
```

Check cluster:

```bash
kubectl cluster-info
kubectl get nodes
```

Expected output:

```bash
NAME                         STATUS   ROLES           AGE
k8s-demo-control-plane      Ready    control-plane   ...
```

# What This Means

## `kind create cluster`

Creates a local Kubernetes cluster using Docker containers.

## `kubectl get nodes`

Shows Kubernetes nodes available in cluster.

# Step 2: Install Dependencies

```bash
npm install
```

# Why This Is Needed

Installs Node.js dependencies from `package.json`.

Without this:

- Express server cannot run
- Application will fail to start

# Step 3: Run Application Locally

```bash
node server.js
```

Open:

```text
http://localhost:3000
```

# Why Run Locally First?

This verifies:

- app works correctly
- no syntax errors
- no runtime crashes

Always test locally before Docker or Kubernetes.

# Step 4: Build Docker Image

```bash
docker build -t k8s-demo:v1 .
```

# What This Means

## `docker build`

Creates Docker image from Dockerfile.

## `-t k8s-demo:v1`

Tags image.

- `k8s-demo` = image name
- `v1` = version

## `.`

Uses current folder as build context.

# Step 5: Test Docker Container

```bash
docker run -p 3000:3000 k8s-demo:v1
```

Open:

```text
http://localhost:3000
```

# Why Test Docker Container?

Before Kubernetes, verify container works properly.

This avoids debugging Kubernetes when actual issue is container itself.

# Step 6: Load Image into kind

```bash
kind load docker-image k8s-demo:v1 --name k8s-demo
```

# Why This Is Required

kind nodes run inside Docker containers.

Your local Docker images are NOT automatically available inside Kubernetes cluster.

Without loading image:

Pods may fail with:

```text
ImagePullBackOff
```

because Kubernetes tries pulling image from Docker Hub.

# Step 7: Deploy Application

```bash
kubectl apply -f deployment.yaml
```

# What Is deployment.yaml?

Defines:

- container image
- replicas
- ports
- labels
- pod template

# Why Use Deployments?

Deployments provide:

- self-healing
- scaling
- rolling updates
- automatic pod recovery

# Step 8: Check Pods

```bash
kubectl get pods
```

Expected output:

```bash
NAME                        READY   STATUS
k8s-demo-xxxxx             1/1     Running
```

# What Are Pods?

Pods are smallest deployable unit in Kubernetes.

Each pod contains:

- one or more containers

# Step 9: Create Service

```bash
kubectl apply -f service.yaml
```

# What Is a Service?

Pods are temporary.

Their IP addresses change.

Services provide:

- stable networking
- internal communication
- load balancing

Without services:

- pod IP changes break application access

# Recommended service.yaml for kind

```yaml
apiVersion: v1
kind: Service

metadata:
  name: k8s-demo-service

spec:
  selector:
    app: k8s-demo

  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000

  type: ClusterIP
```

# Step 10: Access Application

Use port forwarding:

```bash
kubectl port-forward service/k8s-demo-service 8080:80
```

Open:

```text
http://localhost:8080
```

# Why Use Port Forward?

kind networking is container-based.

For beginners:

```bash
kubectl port-forward
```

is easiest and most reliable.

# Kubernetes Commands Explained

# Get Pods

```bash
kubectl get pods
```

Shows running pods.

# Get All Resources

```bash
kubectl get all
```

Shows:

- pods
- services
- deployments
- replica sets

Useful for cluster overview.

# Describe Pod

```bash
kubectl describe pod POD_NAME
```

Shows:

- events
- scheduling issues
- image errors
- crash information

Most useful debugging command.

# View Logs

```bash
kubectl logs POD_NAME
```

Shows application logs inside container.

Useful for:

- crashes
- runtime errors
- debugging

# Execute Inside Container

```bash
kubectl exec -it POD_NAME -- sh
```

Opens shell inside container.

Useful for:

- checking files
- debugging
- testing connectivity

# Scale Deployment

```bash
kubectl scale deployment k8s-demo --replicas=5
```

Creates additional pods automatically.

# Delete Pod

```bash
kubectl delete pod POD_NAME
```

Kubernetes automatically recreates pod.

Demonstrates:

- self-healing
- reconciliation loop

# Restart Deployment

```bash
kubectl rollout restart deployment k8s-demo
```

Restarts all pods safely.

Useful after config changes.

# Common Errors

# ImagePullBackOff

## Cause

Kubernetes cannot access image.

## Fix

```bash
kind load docker-image k8s-demo:v1 --name k8s-demo
```

# CrashLoopBackOff

## Cause

Application crashes repeatedly.

## Debug

```bash
kubectl logs POD_NAME
```

# Pod Stuck in Pending

## Cause

Cluster lacks resources or scheduling failed.

## Debug

```bash
kubectl describe pod POD_NAME
```

# Important Beginner Mistake

If you rebuild image:

```bash
docker build -t k8s-demo:v1 .
```

You MUST reload image again:

```bash
kind load docker-image k8s-demo:v1 --name k8s-demo
```

Otherwise Kubernetes still uses old image version inside kind cluster.

# Delete Cluster

```bash
kind delete cluster --name k8s-demo
```

# Multi-Node Cluster

Create config file:

## kind-config.yaml

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Create cluster:

```bash
kind create cluster --name k8s-demo --config kind-config.yaml
```

Check nodes:

```bash
kubectl get nodes
```

# Dockerfile Explanation

```dockerfile
FROM node:18-alpine
```

Uses lightweight Node.js image.

```dockerfile
WORKDIR /app
```

Sets working directory.

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

Documents application port.

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

Deployment identifies pods using labels.

```yaml
labels:
  app: k8s-demo
```

Labels organize Kubernetes resources.

```yaml
image: k8s-demo:v1
```

Container image to run.

```yaml
containerPort: 3000
```

Application listens on port 3000.

# Real Kubernetes Concepts Learned

| Concept      | Meaning                        |
| ------------ | ------------------------------ |
| Pod          | Running container instance     |
| Deployment   | Manages pod lifecycle          |
| Service      | Stable networking layer        |
| ReplicaSet   | Maintains desired pod count    |
| Labels       | Resource identification        |
| Selectors    | Connect resources together     |
| Scaling      | Increase/decrease pod count    |
| Self-healing | Auto recreation of failed pods |

# Useful Resources

- [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)
- [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/)
- [https://docs.docker.com/](https://docs.docker.com/)
