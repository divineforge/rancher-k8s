# MacBook M1 Pro Setup

## Overview

Run Kubernetes and Rancher locally on your MacBook M1 Pro. Perfect for learning, development, and testing without cloud costs.

**Time Required:** 30-60 minutes
**Cost:** Free (uses local resources)
**Hardware:** MacBook M1/M2/M3 Pro with 16GB+ RAM recommended

---

## Architecture Options for M1 Macs

### Option 1: Rancher Desktop (Recommended for Beginners)
**What:** All-in-one Kubernetes + container runtime
**Pros:** Simplest setup, GUI, includes Rancher dashboard
**Cons:** Resource-heavy, opinionated

### Option 2: Docker Desktop + K3s
**What:** Docker Desktop with K3s cluster
**Pros:** Most compatible, well-documented
**Cons:** Docker Desktop licensing for enterprise use

### Option 3: Colima + K3s (Lightweight)
**What:** Open-source Docker Desktop alternative
**Pros:** Free, lightweight, fast
**Cons:** Less GUI support

### Option 4: Multipass + K3s
**What:** Canonical's VM manager with K3s
**Pros:** Closest to real infrastructure
**Cons:** VM overhead

---

## Option 1: Rancher Desktop (Easiest Start)

### Installation

**Step 1: Download Rancher Desktop**

```bash
# Using Homebrew (recommended)
brew install --cask rancher-desktop

# Or download from:
# https://rancherdesktop.io/
```

**Step 2: Launch Rancher Desktop**

1. Open Rancher Desktop from Applications
2. On first launch, choose:
   - **Container Runtime:** containerd (recommended) or dockerd
   - **Kubernetes Version:** Latest stable (e.g., v1.28.x)
   - **Memory:** 6GB (8GB if you have 16GB RAM)
   - **CPUs:** 4 cores

3. Wait 2-3 minutes for Kubernetes to start
4. You'll see "Kubernetes is running" in the UI

**Step 3: Verify Installation**

```bash
# Check Rancher Desktop provides kubectl
kubectl version

# Check nodes (you'll see 1 local node)
kubectl get nodes

# Expected output:
# NAME                   STATUS   ROLE    AGE   VERSION
# lima-rancher-desktop   Ready    <all>   1m    v1.28.x
```

**Step 4: Access Rancher Dashboard**

Rancher Desktop includes a local Rancher Manager instance:

1. In Rancher Desktop UI, go to **"Preferences"**
2. Enable **"Rancher Dashboard"** (if not already enabled)
3. Click **"Launch Rancher"** or go to `http://localhost:PORT`
4. Set admin password on first access

**What you have now:** A single-node Kubernetes cluster with Rancher UI running locally

---

### What You Can Do with Rancher Desktop

#### Deploy Sample Applications

```bash
# Create a namespace
kubectl create namespace demo

# Deploy nginx
kubectl create deployment nginx --image=nginx --namespace demo

# Expose as service
kubectl expose deployment nginx --port=80 --type=LoadBalancer --namespace demo

# Get service URL
kubectl get svc -n demo

# Access nginx
open http://localhost:<PORT>
```

#### Use Helm Charts

```bash
# Rancher Desktop includes Helm
helm version

# Add Bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# Install WordPress
helm install my-wordpress bitnami/wordpress --namespace demo
```

#### Test Rancher Features

1. **App Catalog:** Deploy apps from Rancher UI
2. **Projects:** Create projects and organize namespaces
3. **Monitoring:** Install Rancher Monitoring (Prometheus + Grafana)
4. **Logging:** Test log aggregation

---

### Limitations

- ❌ Single-node only (can't test HA)
- ❌ Limited to ~8GB memory for Kubernetes
- ❌ No multi-cluster management (only local cluster)
- ⚠️ Performance not representative of production

**Best for:** Learning Kubernetes basics, testing Helm charts, UI exploration

---

## Option 2: Docker Desktop + K3s (Most Compatible)

### Installation

**Step 1: Install Docker Desktop**

```bash
# Download from:
# https://www.docker.com/products/docker-desktop/

# Or via Homebrew
brew install --cask docker
```

Launch Docker Desktop and enable Kubernetes:
1. Docker Desktop → Settings → Kubernetes
2. Check "Enable Kubernetes"
3. Apply & Restart (takes 3-5 minutes)

**Step 2: Verify Kubernetes**

```bash
kubectl config current-context
# Should show: docker-desktop

kubectl get nodes
# Shows 1 node named docker-desktop
```

**Step 3: Install Rancher (Optional)**

Run Rancher Server in Docker:

```bash
# Run Rancher container
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  --name rancher \
  rancher/rancher:latest

# Wait 2-3 minutes, then access:
open https://localhost

# Get bootstrap password
docker logs rancher 2>&1 | grep "Bootstrap Password:"
```

**Note:** Accept the self-signed certificate warning in your browser

---

### What You Can Learn

- ✅ Kubernetes fundamentals (pods, services, deployments)
- ✅ kubectl commands
- ✅ Helm chart deployments
- ✅ Rancher UI (if you install Rancher container)
- ✅ Docker workflow integration

---

### Troubleshooting Docker Desktop

**Issue:** "Kubernetes is starting" stuck

**Fix:**
```bash
# Reset Kubernetes cluster
# Docker Desktop → Settings → Kubernetes → Reset Kubernetes Cluster
```

**Issue:** High CPU/memory usage

**Fix:**
```bash
# Reduce resources in Docker Desktop settings
# Settings → Resources
# Set CPUs to 4, Memory to 6GB
```

---

## Option 3: Colima + K3s (Lightweight & Free)

Colima is an open-source Docker Desktop alternative, perfect for M1 Macs.

### Installation

**Step 1: Install Colima**

```bash
# Install Colima and Docker CLI
brew install colima docker kubectl

# Install K3s
brew install k3s
```

**Step 2: Start Colima with Kubernetes**

```bash
# Start Colima with K3s
colima start --kubernetes \
  --cpu 4 \
  --memory 8 \
  --disk 50

# This creates a Lima VM with K3s running inside
```

**Step 3: Verify**

```bash
kubectl get nodes
# Shows colima node

docker ps
# Docker commands work via Colima
```

**Step 4: Install Rancher (Optional)**

```bash
# Deploy Rancher using Helm
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system

helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.local \
  --set replicas=1 \
  --set bootstrapPassword=admin

# Wait for Rancher to be ready
kubectl -n cattle-system rollout status deploy/rancher

# Get Rancher URL
kubectl -n cattle-system get ingress
# Access at https://rancher.local (add to /etc/hosts)
```

**Add to /etc/hosts:**
```bash
# Get Colima IP
colima status

# Add to /etc/hosts
sudo nano /etc/hosts
# Add line: 127.0.0.1 rancher.local
```

Access: `https://rancher.local`

---

### Why Choose Colima?

- ✅ **Lightweight:** 2-3GB RAM vs 5-6GB for Docker Desktop
- ✅ **Free:** No enterprise licensing concerns
- ✅ **Fast startup:** ~30 seconds vs 2+ minutes for Docker Desktop
- ✅ **Native ARM support:** Built for M1/M2/M3
- ⚠️ **Less GUI:** Command-line focused

**Best for:** Developers who prefer CLI, want to avoid Docker Desktop licensing

---

## Option 4: Multipass + K3s (Most Production-Like)

### Installation

**Step 1: Install Multipass**

```bash
brew install --cask multipass
```

**Step 2: Create VM with K3s**

```bash
# Launch Ubuntu VM
multipass launch --name rancher-node \
  --cpus 4 \
  --memory 8G \
  --disk 50G \
  22.04

# Shell into VM
multipass shell rancher-node

# Install K3s inside VM
curl -sfL https://get.k3s.io | sh -

# Exit VM
exit
```

**Step 3: Get kubeconfig**

```bash
# Copy kubeconfig from VM
multipass exec rancher-node -- sudo cat /etc/rancher/k3s/k3s.yaml > k3s.yaml

# Get VM IP
VM_IP=$(multipass info rancher-node | grep IPv4 | awk '{print $2}')

# Replace 127.0.0.1 with VM IP
sed -i '' "s/127.0.0.1/$VM_IP/" k3s.yaml

# Set kubeconfig
export KUBECONFIG=$(pwd)/k3s.yaml

# Test
kubectl get nodes
```

**Step 4: Install Rancher**

```bash
# From your Mac (not inside VM)
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

kubectl create namespace cattle-system
kubectl create namespace cert-manager

# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml

# Wait for cert-manager
kubectl -n cert-manager rollout status deploy/cert-manager

# Install Rancher
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.local \
  --set replicas=1 \
  --set bootstrapPassword=admin

# Wait for Rancher
kubectl -n cattle-system rollout status deploy/rancher
```

**Step 5: Access Rancher**

```bash
# Port forward (or add to /etc/hosts)
kubectl -n cattle-system port-forward svc/rancher 8443:443

# Open browser
open https://localhost:8443
```

---

### Why Choose Multipass?

- ✅ **Most realistic:** Separate VM mimics real infrastructure
- ✅ **Multi-node capable:** Can create multiple VMs for HA testing
- ✅ **Snapshot support:** Save VM states
- ✅ **Clean isolation:** Doesn't interfere with Docker Desktop
- ⚠️ **More complex:** Requires VM management knowledge

**Best for:** Learning production-like deployments, testing HA configurations

---

## Performance Comparison

| Method | RAM Usage | CPU Usage | Startup Time | Complexity |
|--------|-----------|-----------|--------------|------------|
| **Rancher Desktop** | 5-6GB | Medium | 2-3 min | Low |
| **Docker Desktop** | 4-5GB | Medium | 2-3 min | Low |
| **Colima** | 2-3GB | Low | 30 sec | Medium |
| **Multipass** | 6-8GB | Medium | 1-2 min | High |

**Recommendation for M1 Pro:**
- **16GB RAM:** Any option works, Rancher Desktop easiest
- **8GB RAM:** Use Colima (lightest footprint)

---

## What You Can Learn on MacBook M1

### ✅ Kubernetes Fundamentals
- Pods, services, deployments, ConfigMaps, secrets
- kubectl commands
- YAML manifest creation
- Debugging (logs, exec, describe)

### ✅ Application Deployment
- Containerize applications
- Deploy to Kubernetes
- Test CI/CD workflows
- Helm chart development

### ✅ Rancher Features (if Rancher installed)
- UI navigation
- App catalog usage
- Project/namespace management
- Basic RBAC

### ✅ Development Workflow
- Local inner loop (code → build → deploy → test)
- Hot reload setups
- Debugging in Kubernetes

---

## What You Can't Learn

### ❌ High Availability
- Multi-master setups
- etcd clustering
- Load balancer configurations

### ❌ Multi-Cluster Management
- Rancher managing multiple clusters
- Cross-cluster app deployment
- Global DNS

### ❌ Production Operations
- Real backup/restore scenarios
- Node failure handling
- Cluster upgrades at scale

### ❌ Enterprise Integration
- Active Directory integration
- Enterprise monitoring tools
- Physical network policies

**Verdict:** Great for 80% of learning, but need real infrastructure for production validation

---

## Best Practices for M1 Learning

### 1. Resource Management

```bash
# Don't run too many pods
kubectl get pods --all-namespaces | wc -l
# Keep under 50 for smooth performance

# Clean up regularly
kubectl delete namespace <unused-namespaces>
docker system prune -a
```

### 2. Use Local Image Registry

```bash
# Build images locally
docker build -t myapp:latest .

# No need to push to Docker Hub for local testing
kubectl run myapp --image=myapp:latest --image-pull-policy=Never
```

### 3. Optimize for ARM Architecture

```bash
# Use ARM-compatible images
docker pull --platform=linux/arm64 nginx:latest

# Check image architecture
docker inspect nginx:latest | grep Architecture
```

---

## Sample Project: Deploy WordPress on M1

```bash
# 1. Create namespace
kubectl create namespace wordpress

# 2. Deploy MySQL
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: wordpress
type: Opaque
data:
  password: $(echo -n "MyPassword123" | base64)
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: wordpress
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        - name: MYSQL_DATABASE
          value: wordpress
        ports:
        - containerPort: 3306
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: wordpress
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
EOF

# 3. Deploy WordPress
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress
spec:
  type: LoadBalancer
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
EOF

# 4. Access WordPress
kubectl -n wordpress get svc wordpress
# Open http://localhost:<PORT> in browser
```

**What you learned:**
- Secrets management
- Multi-tier application deployment
- Service networking
- LoadBalancer type services

---

## Troubleshooting Common Issues

### Issue: "ImagePullBackOff" errors

**Cause:** Pulling AMD64 images on ARM architecture

**Fix:**
```bash
# Use ARM-compatible images
# Check Docker Hub for arm64 tags
docker pull --platform=linux/arm64 <image>
```

### Issue: Slow performance

**Fix:**
```bash
# Reduce resource limits
# Limit number of running pods
# Increase RAM allocation to Kubernetes
```

### Issue: "connection refused" to services

**Fix:**
```bash
# Use port-forward for debugging
kubectl port-forward svc/<service-name> 8080:80 -n <namespace>

# Check service endpoints
kubectl get endpoints -n <namespace>
```

---

## Next Steps

After mastering local setup:

1. **Import to Rancher Hosted:** Connect your local cluster to [Rancher Hosted Service](rancher-hosted.md)
2. **Deploy to Cloud:** Try [Bare Metal Setup](bare-metal-setup.md) guide
3. **Learn Advanced Topics:** [Security](../security/index.md), [Operations](../operations/index.md)

---

## Resources

- [Rancher Desktop Docs](https://docs.rancherdesktop.io/)
- [Colima GitHub](https://github.com/abiosoft/colima)
- [K3s Documentation](https://docs.k3s.io/)
- [Kubernetes Basics](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

---

**Estimated Learning Time:** 1-2 weeks of daily experimentation
**Cost:** Free (your Mac's resources)
**Best for:** Learning Kubernetes, development workflows, Rancher UI basics

**Status:** Hands-on guide
**Last Updated:** January 2026
