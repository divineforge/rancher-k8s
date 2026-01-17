# Bare Metal Setup Guide

## Overview

Deploy a production-ready Rancher environment on bare metal or VMs for realistic testing and evaluation.

**Time Required:** 2-4 hours (experienced), 1-2 days (learning)
**Infrastructure:** 6 servers/VMs minimum (3 Rancher, 3 workload cluster)
**Skill Level:** Intermediate to Advanced

---

## Architecture Overview

```
┌───────────────────────────────────────────────┐
│      Rancher Management Cluster (HA)          │
│                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Rancher  │  │ Rancher  │  │ Rancher  │   │
│  │  Node 1  │  │  Node 2  │  │  Node 3  │   │
│  │ (Master) │  │ (Master) │  │ (Master) │   │
│  └──────────┘  └──────────┘  └──────────┘   │
│                                               │
└───────────────┬───────────────────────────────┘
                │
                │ Manages
                ↓
┌───────────────────────────────────────────────┐
│     Downstream Workload Cluster               │
│                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Worker  │  │  Worker  │  │  Worker  │   │
│  │  Node 1  │  │  Node 2  │  │  Node 3  │   │
│  └──────────┘  └──────────┘  └──────────┘   │
│                                               │
└───────────────────────────────────────────────┘
```

---

## Prerequisites

### Hardware Requirements

**Rancher Management Cluster (3 nodes):**
- **CPU:** 4 cores per node
- **RAM:** 8GB per node (16GB recommended)
- **Storage:** 50GB SSD per node
- **Network:** 1Gbps minimum

**Downstream Cluster (3+ nodes):**
- **CPU:** 4-8 cores per node
- **RAM:** 16GB per node (32GB recommended)
- **Storage:** 100GB SSD per node
- **Network:** 1Gbps minimum

**Total:** 6 servers/VMs minimum

---

### Software Requirements

**Operating System (choose one):**
- **Ubuntu 22.04 LTS** (recommended for beginners)
- **Rocky Linux 9** (RHEL-compatible, enterprise-friendly)
- **SLES 15** (SUSE Linux, official Rancher support)

**Network:**
- Static IP addresses for all nodes
- DNS resolution between nodes
- NTP synchronized
- Firewall configured (see ports below)

---

### Required Ports

**Rancher Server Nodes:**
- TCP 80, 443 (HTTP/HTTPS)
- TCP 2379-2380 (etcd)
- TCP 6443 (Kubernetes API)
- TCP/UDP 8472 (Flannel VXLAN)
- TCP 10250 (kubelet)

**Downstream Cluster Nodes:**
- TCP 6443 (Kubernetes API)
- TCP 10250 (kubelet)
- TCP/UDP 8472 (Flannel VXLAN)
- Outbound to Rancher server (TCP 443)

---

## Step-by-Step Setup

### Phase 1: Prepare Infrastructure

#### Option A: Using Physical Servers

1. **Rack and cable servers**
2. **Configure BIOS/UEFI:**
   - Enable PXE boot (if using automated provisioning)
   - Disable Secure Boot (or configure shim)
3. **Install OS** (Ubuntu 22.04 recommended)

#### Option B: Using VMs (VirtualBox, VMware, Proxmox)

**Example: Using Multipass (Quick Setup)**

```bash
# Create 3 VMs for Rancher cluster
for i in 1 2 3; do
  multipass launch --name rancher-node-$i \
    --cpus 4 \
    --memory 8G \
    --disk 50G \
    22.04
done

# Create 3 VMs for workload cluster
for i in 1 2 3; do
  multipass launch --name worker-node-$i \
    --cpus 4 \
    --memory 16G \
    --disk 100G \
    22.04
done

# List all VMs
multipass list
```

---

### Phase 2: Configure Operating System

**Run on ALL nodes:**

```bash
# SSH into each node (or use Multipass shell)
# multipass shell rancher-node-1

# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Disable swap (Kubernetes requirement)
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# 3. Set hostname
sudo hostnamectl set-hostname rancher-node-1  # Change per node

# 4. Add hosts entries (adjust IPs)
sudo tee -a /etc/hosts <<EOF
10.0.0.11 rancher-node-1
10.0.0.12 rancher-node-2
10.0.0.13 rancher-node-3
10.0.0.21 worker-node-1
10.0.0.22 worker-node-2
10.0.0.23 worker-node-3
EOF

# 5. Configure firewall
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 6443/tcp # Kubernetes API
sudo ufw allow 2379:2380/tcp  # etcd
sudo ufw allow 10250/tcp  # kubelet
sudo ufw allow 8472/udp   # Flannel
sudo ufw enable

# 6. Install required packages
sudo apt install -y curl wget git vim net-tools

# 7. Sync time
sudo apt install -y chrony
sudo systemctl enable chrony
sudo systemctl start chrony
```

---

### Phase 3: Install RKE2 on Rancher Cluster

RKE2 is Rancher's next-gen Kubernetes distribution, ideal for production.

#### On First Rancher Node (rancher-node-1)

```bash
# 1. Install RKE2 server
curl -sfL https://get.rke2.io | sudo sh -

# 2. Create config file
sudo mkdir -p /etc/rancher/rke2
sudo tee /etc/rancher/rke2/config.yaml <<EOF
token: my-shared-secret-token
tls-san:
  - rancher-node-1
  - 10.0.0.11
cluster-cidr: 10.42.0.0/16
service-cidr: 10.43.0.0/16
EOF

# 3. Enable and start RKE2
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service

# 4. Wait for RKE2 to be ready (2-3 minutes)
sudo systemctl status rke2-server

# 5. Set up kubectl
mkdir ~/.kube
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc

# 6. Add kubectl to PATH
export PATH=$PATH:/var/lib/rancher/rke2/bin
echo 'export PATH=$PATH:/var/lib/rancher/rke2/bin' >> ~/.bashrc

# 7. Verify single-node cluster
kubectl get nodes
```

#### On Additional Rancher Nodes (rancher-node-2 and rancher-node-3)

```bash
# Get the first node's IP
FIRST_NODE_IP="10.0.0.11"

# 1. Install RKE2
curl -sfL https://get.rke2.io | sudo sh -

# 2. Create config to join cluster
sudo mkdir -p /etc/rancher/rke2
sudo tee /etc/rancher/rke2/config.yaml <<EOF
server: https://${FIRST_NODE_IP}:9345
token: my-shared-secret-token
tls-san:
  - rancher-node-2  # Change per node
  - 10.0.0.12       # Change per node
EOF

# 3. Enable and start
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service

# 4. Wait for join (1-2 minutes)
sudo systemctl status rke2-server
```

**Verify 3-node cluster (from rancher-node-1):**

```bash
kubectl get nodes
# Should show 3 nodes in Ready state
```

---

### Phase 4: Install Rancher Management Server

#### Install Helm

```bash
# From rancher-node-1
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

#### Install cert-manager

```bash
# cert-manager manages TLS certificates
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io
helm repo update

kubectl create namespace cert-manager

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.13.3

# Wait for cert-manager to be ready
kubectl -n cert-manager rollout status deploy/cert-manager
```

#### Install Rancher

```bash
# Add Rancher Helm repo
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update

# Create namespace
kubectl create namespace cattle-system

# Install Rancher
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.yourdomain.com \
  --set bootstrapPassword=admin \
  --set replicas=3

# Monitor Rancher deployment
kubectl -n cattle-system rollout status deploy/rancher

# Wait for all pods to be running (3-5 minutes)
kubectl -n cattle-system get pods
```

#### Access Rancher UI

**Option 1: Port Forward (for testing)**

```bash
# From your local machine (not the server)
kubectl port-forward -n cattle-system svc/rancher 8443:443

# Open browser: https://localhost:8443
```

**Option 2: Configure DNS and Load Balancer (production)**

1. Point DNS record `rancher.yourdomain.com` to any of the 3 Rancher nodes
2. Or configure a load balancer (HAProxy, nginx) in front of the 3 nodes
3. Access `https://rancher.yourdomain.com`

**First Login:**
1. Accept self-signed certificate warning
2. Username: `admin`
3. Password: `admin` (or what you set in `bootstrapPassword`)
4. You'll be prompted to change the password
5. Set server URL (e.g., `https://rancher.yourdomain.com`)

---

### Phase 5: Create Downstream Workload Cluster

#### Option A: RKE2 Custom Cluster (Recommended)

**In Rancher UI:**

1. Go to **Cluster Management** → **Create**
2. Select **Custom**
3. Give cluster a name: `production-workload`
4. Select Kubernetes version (latest stable)
5. Click **Create**
6. **Registration Command** appears - copy it

**On Each Worker Node (worker-node-1, worker-node-2, worker-node-3):**

```bash
# Run the registration command from Rancher UI
# It looks like this (your token will be different):
curl --insecure -sfL https://rancher.yourdomain.com/v3/import/abc123xyz... | kubectl apply -f -
```

Wait 2-3 minutes. Cluster will appear as **Active** in Rancher UI.

---

#### Option B: RKE2 Cluster (Manual, then import)

**On First Worker Node:**

```bash
# Install RKE2
curl -sfL https://get.rke2.io | sudo sh -

# Configure
sudo mkdir -p /etc/rancher/rke2
sudo tee /etc/rancher/rke2/config.yaml <<EOF
token: worker-cluster-token
EOF

# Start
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service

# Set up kubectl
mkdir ~/.kube
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config
```

**On Additional Workers:**

```bash
FIRST_WORKER_IP="10.0.0.21"

curl -sfL https://get.rke2.io | sudo INSTALL_RKE2_TYPE="agent" sh -

sudo mkdir -p /etc/rancher/rke2
sudo tee /etc/rancher/rke2/config.yaml <<EOF
server: https://${FIRST_WORKER_IP}:9345
token: worker-cluster-token
EOF

sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```

**Import to Rancher:**
Follow the import steps from Rancher UI.

---

### Phase 6: Deploy Sample Application

#### From Rancher UI

1. Select your workload cluster
2. Go to **Apps** → **Charts**
3. Install **nginx** or **WordPress**
4. Select namespace (or create new)
5. Click **Install**
6. Check **Workloads** → **Deployments** to see status

#### Using kubectl

```bash
# Get kubeconfig for workload cluster from Rancher UI
# Cluster → ... (menu) → Download kubeconfig

# Set KUBECONFIG
export KUBECONFIG=/path/to/workload-cluster.yaml

# Create deployment
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Get service
kubectl get svc nginx
```

---

## Validation Checklist

### Rancher Management Cluster

- [ ] All 3 nodes show as `Ready`
- [ ] Rancher UI accessible
- [ ] cert-manager pods running
- [ ] Rancher pods running (3 replicas)
- [ ] Can create new clusters

### Downstream Cluster

- [ ] All worker nodes show as `Ready`
- [ ] Cluster shows as `Active` in Rancher
- [ ] Can deploy applications
- [ ] Pods can reach internet (if needed)
- [ ] Services are accessible

### Integration

- [ ] Rancher can manage downstream cluster
- [ ] Monitoring stack installable
- [ ] Logging stack installable
- [ ] App catalog works

---

## Troubleshooting

### Issue: Nodes not joining cluster

**Check:**
```bash
# On failing node
sudo systemctl status rke2-server  # or rke2-agent
sudo journalctl -u rke2-server -f

# Common issues:
# - Token mismatch
# - Firewall blocking ports
# - Incorrect server IP
# - Time sync issues
```

---

### Issue: Rancher UI not accessible

**Check:**
```bash
kubectl -n cattle-system get pods
kubectl -n cattle-system logs -l app=rancher

# Common issues:
# - cert-manager not ready
# - DNS not configured
# - Hostname mismatch
```

---

### Issue: Downstream cluster not importing

**Check:**
```bash
# On downstream cluster
kubectl get pods -n cattle-system

# Common issues:
# - Network connectivity to Rancher
# - Certificate issues
# - Agent can't reach Rancher URL
```

---

## Next Steps

### Week 1: Operational Testing
- [ ] Test cluster upgrades
- [ ] Deploy monitoring (Prometheus/Grafana)
- [ ] Deploy logging (Loki or ELK)
- [ ] Test backup/restore

### Week 2: Security Validation
- [ ] Configure AD/LDAP integration
- [ ] Test RBAC policies
- [ ] Implement network policies
- [ ] Configure audit logging

### Week 3: Application Testing
- [ ] Deploy real workloads
- [ ] Test CI/CD integration
- [ ] Performance benchmarking
- [ ] Load testing

---

## Production Checklist

Before going to production:

- [ ] High availability validated (simulate node failures)
- [ ] Backup/restore procedures tested
- [ ] Monitoring and alerting configured
- [ ] Security hardening completed (CIS benchmark)
- [ ] Disaster recovery plan documented
- [ ] Team training completed
- [ ] InfoSec approval obtained
- [ ] Support contract in place (if needed)

---

## Cost Estimate

### Infrastructure Costs (3 years)

**Hardware (6 servers, mid-range):**
- Servers: $60,000
- Network: $10,000
- Storage: $15,000
- **Total:** $85,000

**Operational:**
- Power: $3,000/year
- Cooling: $2,000/year
- Maintenance: $5,000/year
- **Total:** $30,000 (3 years)

**Software:**
- Rancher: Free (open source) or $X/node (Prime)
- OS: Free (Ubuntu/Rocky) or $Y/node (SLES)

**Grand Total:** ~$115,000 - $150,000 (3 years)

Compare with cloud: AWS/GKE equivalent = $400,000+ (3 years)

---

## Additional Resources

- [RKE2 Documentation](https://docs.rke2.io/)
- [Rancher Installation Guide](https://ranchermanager.docs.rancher.com/pages-for-subheaders/installation-and-upgrade)
- [Bare Metal Architecture](../architecture/bare-metal.md)
- [Day 2 Operations](../operations/day2-ops.md)

---

**Estimated Setup Time:** 4-8 hours (experienced), 1-2 days (learning)
**Difficulty:** Intermediate to Advanced
**Best for:** Production validation, full-stack learning

**Status:** Production setup guide
**Last Updated:** January 2026
