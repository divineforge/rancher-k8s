# Rancher Hosted Service (Quick Start)

## Overview

The fastest way to start with Rancher - no infrastructure required. Perfect for exploring the UI and understanding multi-cluster management.

**Time Required:** 5-10 minutes
**Cost:** Free trial available
**Skill Level:** Beginner

---

## What is Rancher Hosted Service?

SUSE offers a cloud-hosted Rancher management server at **cloud.rancher.com** (previously known as Rancher Prime Hosted).

### What You Get

- ✅ Fully managed Rancher Server (no installation needed)
- ✅ Highly available (SUSE manages infrastructure)
- ✅ Automatic updates
- ✅ Multi-cluster management UI
- ✅ Free trial for evaluation

### What You Don't Get

- ❌ You don't control the Rancher Server infrastructure
- ❌ You still need to bring your own Kubernetes clusters
- ❌ Limited to import/register existing clusters (can't provision from scratch)

**Think of it as:** Rancher-as-a-Service

---

## Step-by-Step Setup

### Step 1: Sign Up

1. Visit **https://www.rancher.com/products/rancher-hosted**
2. Click **"Try Rancher Prime Hosted"**
3. Sign up with:
   - Email address
   - GitHub account
   - Google account

**Note:** Check if trial is available, or contact SUSE sales for evaluation access

---

### Step 2: Access Rancher UI

Once signed up:

1. Log in to **https://cloud.rancher.com** (or your provided URL)
2. You'll see the Rancher Dashboard
3. Default view: **Cluster Management**

**What you're seeing:** A hosted Rancher management server with no downstream clusters yet

---

### Step 3: Create a Test Cluster (Options)

Since Rancher Hosted only **manages** clusters, you need to bring your own Kubernetes cluster to import. Here are quick options:

#### Option A: Use a Cloud Provider Free Tier

**Google Kubernetes Engine (GKE) - Free tier:**
```bash
# Create GKE cluster (free tier: 1 zonal cluster)
gcloud container clusters create rancher-demo \
  --zone us-central1-a \
  --num-nodes 1 \
  --machine-type e2-medium

# Get credentials
gcloud container clusters get-credentials rancher-demo --zone us-central1-a
```

**Amazon EKS - Free tier (EC2 costs apply):**
```bash
# Using eksctl
eksctl create cluster \
  --name rancher-demo \
  --region us-west-2 \
  --nodes 1 \
  --node-type t3.medium
```

**Azure AKS - Free tier (node costs apply):**
```bash
az aks create \
  --resource-group rancher-demo-rg \
  --name rancher-demo \
  --node-count 1 \
  --node-vm-size Standard_B2s
```

---

#### Option B: Use K3s on a VM

**DigitalOcean Droplet ($4/month):**

```bash
# SSH into droplet
ssh root@your-droplet-ip

# Install K3s
curl -sfL https://get.k3s.io | sh -

# Get kubeconfig
cat /etc/rancher/k3s/k3s.yaml
```

Copy the kubeconfig to your local machine and replace `127.0.0.1` with the droplet's IP.

---

#### Option C: Use Minikube/K3s Locally (For Testing)

**Note:** Local clusters can't be accessed by Rancher Hosted directly unless you expose them via tunneling (not recommended for evaluation)

**Better approach:** Use this to learn kubectl, then import a cloud cluster to Rancher Hosted

---

### Step 4: Import Cluster to Rancher

Once you have a Kubernetes cluster:

1. **In Rancher UI:** Click **"Cluster Management"**
2. Click **"Import Existing"**
3. Give your cluster a name (e.g., "demo-cluster")
4. Click **"Create"**
5. **Copy the kubectl command** shown (looks like this):

```bash
kubectl apply -f https://cloud.rancher.com/v3/import/abc123xyz...
```

6. **Run the command** on your Kubernetes cluster (where you have kubectl access)
7. Wait 30-60 seconds
8. Cluster appears in Rancher UI with status **"Active"**

**What just happened:** Rancher installed an agent on your cluster that communicates back to the hosted Rancher server

---

### Step 5: Explore Rancher Features

Now that your cluster is connected, explore:

#### **Cluster Dashboard**
- Click on your cluster name
- View: Nodes, pods, deployments, events
- Real-time metrics (if metrics server is installed)

#### **App Catalog**
- Go to **Apps → Charts**
- Try deploying sample apps:
  - **nginx** (web server)
  - **redis** (cache)
  - **wordpress** (CMS)

**Example: Deploy nginx:**
1. Apps → Charts → nginx
2. Click "Install"
3. Select namespace (create new: "web-apps")
4. Click "Install"
5. Check **Workloads → Deployments** to see it running

#### **User Management**
- Go to **Users & Authentication**
- Add users (if evaluating multi-tenancy)
- Test different permission levels

#### **Projects & Namespaces**
- Create a **Project** (e.g., "Development")
- Projects group multiple namespaces
- Assign users to projects with roles

---

## What You Can Learn

### ✅ Rancher UI Exploration
- Multi-cluster dashboard
- Workload management
- Monitoring and logging integration
- RBAC and user management

### ✅ Multi-Cluster Management
- Import multiple clusters (cloud + on-prem)
- Consistent RBAC across clusters
- Centralized app catalog

### ✅ GitOps with Fleet
- Deploy apps via Git repositories
- Multi-cluster deployments
- Continuous reconciliation

---

## What You Can't Learn (Limitations)

### ❌ Rancher Server Management
- You don't install or maintain Rancher
- Can't customize server configuration
- Limited to features SUSE enables

### ❌ Full Infrastructure Control
- Can't test HA configurations for Rancher
- Can't test backup/restore of Rancher server
- Can't test air-gapped installations

### ❌ Advanced Networking
- No control over Rancher's network configuration
- Can't test custom load balancers
- Firewall rules are managed by SUSE

**Verdict:** Great for UI evaluation, not for production validation

---

## Cost Considerations

### Free Trial
- Typically 30-60 days
- Check with SUSE for current offerings
- May require credit card for verification

### Paid Plans
- Pricing based on number of nodes managed
- Support tier included
- Contact SUSE sales for enterprise pricing

**Tip:** Use free trial to evaluate, then decide between hosted vs self-hosted for production

---

## Banking/Enterprise Considerations

### ✅ Pros
- SUSE manages security patches
- High availability included
- No ops burden for Rancher server
- Compliance (check SUSE's certifications)

### ⚠️ Cons
- Data flows through SUSE infrastructure
- Dependency on SUSE uptime
- May not meet data sovereignty requirements
- Limited control for audits

**Recommendation:** Good for dev/staging, likely not approved for production banking workloads (check with InfoSec)

---

## Troubleshooting

### Cluster Import Fails

**Error:** "Cluster not reachable"

**Fix:**
- Ensure your cluster has outbound internet access
- Check firewall allows traffic to `*.rancher.com`
- Verify kubectl command was run on correct cluster

---

### Agent Not Connecting

**Error:** "Waiting for agent to connect"

**Fix:**
```bash
# Check agent pods on your cluster
kubectl get pods -n cattle-system

# Check agent logs
kubectl logs -n cattle-system -l app=cattle-cluster-agent
```

Common issues:
- Network connectivity
- Certificate problems
- Insufficient permissions

---

## Next Steps After Hosted Service

Once comfortable with Rancher UI:

1. **Try local development:** [MacBook M1 Setup](macbook-m1.md)
2. **Deploy production test:** [Bare Metal Setup](bare-metal-setup.md)
3. **Learn architecture:** [Rancher Components](../architecture/rancher-components.md)

---

## Additional Resources

- [Rancher Prime Hosted Documentation](https://ranchermanager.docs.rancher.com/)
- [Import Existing Cluster Guide](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/kubernetes-clusters-in-rancher-setup/register-existing-clusters)
- [SUSE Support Portal](https://www.suse.com/support/)

---

**Estimated Learning Time:** 2-4 hours of hands-on exploration
**Cost:** Free (trial) + cloud cluster costs (~$5-20/month)
**Best for:** UI familiarization, not production validation

**Status:** Quick start guide
**Last Updated:** January 2026
