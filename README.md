# Kubernetes Cluster Setup & Container Recovery Testing Guide

This beginner-friendly guide will walk you through setting up a Kubernetes cluster, deploying an application, and testing Kubernetes' self-healing capabilities by terminating a container.

## Prerequisites

Before starting, ensure you have:
- A computer with at least 4GB RAM and 2 CPU cores
- Basic command line knowledge
- Administrator/sudo access on your machine

## Step 1: Install Required Tools

### 1.1 Install Docker
```bash
# For Ubuntu/Debian
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER

# For macOS (using Homebrew)
brew install --cask docker

# For Windows
# Download Docker Desktop from docker.com
```

### 1.2 Install kubectl (Kubernetes CLI)
```bash
# For Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# For macOS
brew install kubectl

# For Windows (using Chocolatey)
choco install kubernetes-cli
```

### 1.3 Install Minikube (Local Kubernetes Cluster)
```bash
# For Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# For macOS
brew install minikube

# For Windows
choco install minikube
```

## Step 2: Start Your Kubernetes Cluster

### 2.1 Start Minikube
```bash
# Start minikube with Docker driver
minikube start --driver=docker

# Verify cluster is running
kubectl cluster-info
```

### 2.2 Verify Installation
```bash
# Check nodes
kubectl get nodes

# Check system pods
kubectl get pods -A
```

## Step 3: Create Your Test Application

### 3.1 Create Project Directory
```bash
mkdir k8s-recovery-test
cd k8s-recovery-test
```

### 3.2 Create Deployment YAML
Create a file named `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

## Step 4: Deploy Your Application

### 4.1 Apply the Deployment
```bash
# Deploy the application
kubectl apply -f nginx-deployment.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get services
```

### 4.2 Check Pod Status
```bash
# Watch pods in real-time
kubectl get pods -w

# Get detailed pod information
kubectl get pods -o wide
```

## Step 5: Test Container Recovery

### 5.1 Identify Running Pods
```bash
# List all nginx pods
kubectl get pods -l app=nginx

# Note down one pod name for testing
```

### 5.2 Delete a Pod (Simulate Container Failure)
```bash
# Replace <pod-name> with actual pod name
kubectl delete pod <pod-name>

# Example:
# kubectl delete pod nginx-deployment-7d64c5d6f5-abc12
```

### 5.3 Observe Recovery
```bash
# Watch Kubernetes recreate the pod
kubectl get pods -w

# Check that we still have 3 replicas
kubectl get pods -l app=nginx
```

## Step 6: Advanced Testing - Force Pod Termination

### 6.1 Simulate Unresponsive Container
```bash
# Get into a pod and kill the main process
kubectl exec -it <pod-name> -- /bin/bash
# Inside the pod, run: kill 1

# Or force delete from outside
kubectl delete pod <pod-name> --grace-period=0 --force
```

### 6.2 Monitor Recovery Metrics
```bash
# Check deployment status
kubectl rollout status deployment/nginx-deployment

# View events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check replica set
kubectl get rs
```

## Step 7: Document Your Project on GitHub

### 7.1 Initialize Git Repository
```bash
# Initialize repository
git init

# Create .gitignore file
echo "*.log" > .gitignore
echo ".DS_Store" >> .gitignore
```

### 7.2 Create Documentation Files

Create `README.md`:
```markdown
# Kubernetes Container Recovery Testing Project

## Overview
This project demonstrates Kubernetes' self-healing capabilities by setting up a cluster and testing container recovery.

## Architecture
- **Cluster**: Minikube (local development)
- **Application**: Nginx web server
- **Replicas**: 3 pods for high availability
- **Service**: LoadBalancer for external access

## Quick Start
1. Install prerequisites (Docker, kubectl, minikube)
2. Start cluster: `minikube start`
3. Deploy application: `kubectl apply -f nginx-deployment.yaml`
4. Test recovery: `kubectl delete pod <pod-name>`

## Test Results
Document your observations here:
- Time to detect pod failure: X seconds
- Time to start replacement pod: Y seconds
- Total recovery time: Z seconds

## Commands Used
```bash
# Cluster management
minikube start
kubectl cluster-info

# Application deployment
kubectl apply -f nginx-deployment.yaml
kubectl get pods

# Recovery testing
kubectl delete pod <pod-name>
kubectl get pods -w
```

## Files
- `nginx-deployment.yaml`: Kubernetes deployment configuration
- `test-results.md`: Documentation of test outcomes
- `screenshots/`: Visual documentation of the process
```

### 7.3 Create Test Results Documentation
Create `test-results.md`:
```markdown
# Container Recovery Test Results

## Test Environment
- **Date**: [Current Date]
- **Kubernetes Version**: [Version]
- **Cluster Type**: Minikube
- **Node Count**: 1

## Test Scenarios

### Scenario 1: Graceful Pod Deletion
- **Command**: `kubectl delete pod nginx-deployment-xxx`
- **Expected**: Pod recreated automatically
- **Result**: ✅ Success
- **Recovery Time**: [X seconds]

### Scenario 2: Force Pod Termination
- **Command**: `kubectl delete pod nginx-deployment-xxx --grace-period=0 --force`
- **Expected**: Pod recreated automatically
- **Result**: ✅ Success
- **Recovery Time**: [Y seconds]

## Observations
- Kubernetes maintained desired replica count
- Service remained available during recovery
- New pods received different names but same labels

## Screenshots
Include screenshots of:
- Initial pod status
- Pod deletion command
- Recovery process
- Final pod status
```

### 7.4 Commit and Push to GitHub
```bash
# Add files
git add .

# Commit changes
git commit -m "Initial commit: Kubernetes recovery testing project"

# Create GitHub repository (via web interface)
# Then add remote and push
git remote add origin https://github.com/yourusername/k8s-recovery-test.git
git branch -M main
git push -u origin main
```

## Step 8: Cleanup Resources

When you're done testing:
```bash
# Delete deployment
kubectl delete -f nginx-deployment.yaml

# Stop minikube
minikube stop

# Delete minikube cluster (optional)
minikube delete
```

## Key Concepts Demonstrated

1. **Self-Healing**: Kubernetes automatically replaces failed containers
2. **Desired State**: The cluster maintains the specified number of replicas
3. **High Availability**: Services remain accessible during pod recovery
4. **Monitoring**: Various commands to observe cluster state and events

## Troubleshooting Tips

- If pods don't start: Check resource constraints and node capacity
- If service is unreachable: Verify service configuration and port forwarding
- If recovery is slow: Check cluster resources and event logs

## Next Steps

Consider exploring:
- Rolling updates and rollbacks
- Health checks and readiness probes
- Horizontal Pod Autoscaling
- Persistent volumes and storage
- Multi-node clusters

This project provides a solid foundation for understanding Kubernetes' resilience features and serves as documentation for your learning journey!
