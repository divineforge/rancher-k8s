# Bare Metal Management

## Overview

Bare metal Kubernetes deployments provide maximum performance and control but require careful infrastructure planning. This document covers strategies for managing Kubernetes on bare metal servers with Rancher.

## Why Bare Metal?

### Advantages

**Performance**
- No hypervisor overhead (5-10% performance gain)
- Direct hardware access for GPU/FPGA workloads
- Predictable latency for latency-sensitive applications

**Cost Efficiency**
- Better resource utilization
- No virtualization licensing costs
- Lower infrastructure overhead

**Banking Requirements**
- Data residency compliance
- Physical security control
- Regulatory requirements for on-premise hosting

### Challenges

**Operational Complexity**
- Manual hardware provisioning
- Firmware and driver management
- Physical maintenance overhead

**Inflexibility**
- Harder to resize/repurpose
- Longer provisioning times
- Capacity planning critical

## Bare Metal Kubernetes Options

### 1. RKE2 (Rancher Kubernetes Engine 2)

**What it is:** CNCF-certified Kubernetes distribution optimized for security and simplicity

**Best for:** Production bare metal deployments with Rancher management

**Features:**
- SELinux enforcement by default
- CIS hardened configuration
- Embedded etcd or external etcd support
- Minimal dependencies

**Installation:**
```bash
# Install RKE2 on bare metal node
curl -sfL https://get.rke2.io | sh -

# Enable and start
systemctl enable rke2-server.service
systemctl start rke2-server.service

# Get kubeconfig
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
```

---

### 2. K3s (Lightweight Kubernetes)

**What it is:** Lightweight Kubernetes distribution, perfect for edge and development

**Best for:** Development clusters, edge locations, resource-constrained environments

**Features:**
- Single binary (~60MB)
- Minimal memory footprint
- Built-in SQLite (no etcd required)
- Works great on ARM (M1 Macs!)

**Installation:**
```bash
# Install K3s (works on Linux and macOS via Docker)
curl -sfL https://get.k3s.io | sh -

# Access cluster
sudo k3s kubectl get nodes
```

---

### 3. Kubeadm (Vanilla Kubernetes)

**What it is:** Official Kubernetes bootstrapping tool

**Best for:** Custom configurations, learning Kubernetes internals

**Features:**
- Pure upstream Kubernetes
- Full control over components
- Manual but transparent

---

## Bare Metal Infrastructure Requirements

### Hardware Specifications

#### Rancher Management Cluster
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Nodes** | 3 | 3-5 (HA) |
| **CPU** | 4 cores | 8 cores |
| **RAM** | 8GB | 16GB |
| **Storage** | 50GB SSD | 100GB NVMe |
| **Network** | 1Gbps | 10Gbps |

#### Downstream Workload Clusters
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **Nodes** | 3 | 5+ (scaling) |
| **CPU** | 8 cores | 16+ cores |
| **RAM** | 32GB | 64GB+ |
| **Storage** | 100GB SSD | 500GB NVMe |
| **Network** | 1Gbps | 10Gbps bonded |

### Operating System

**Recommended:**
- **Ubuntu 22.04 LTS** - Best Kubernetes support, long-term stability
- **Rocky Linux 9** - RHEL clone, enterprise-friendly
- **SLES 15** - SUSE Linux, official Rancher support

**Kernel Requirements:**
- Kernel 5.x or newer
- OverlayFS support
- Netfilter/iptables modules

---

## Network Considerations

### Physical Network Design

```
┌─────────────────────────────────────────────────┐
│          Management Network (VLAN 100)          │
│         10.10.100.0/24 - Rancher Server         │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────┐
│         Cluster Network (VLAN 200)              │
│     10.10.200.0/24 - Node-to-Node Traffic       │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────┐
│          Storage Network (VLAN 300)             │
│       10.10.300.0/24 - Persistent Storage       │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────┐
│        Application Network (VLAN 400)           │
│      10.10.400.0/24 - External Access           │
└─────────────────────────────────────────────────┘
```

### Network Requirements

**Bandwidth:**
- 1Gbps minimum for control plane
- 10Gbps recommended for workload nodes
- 25Gbps+ for storage-intensive workloads

**Latency:**
- <1ms between control plane nodes (etcd requirement)
- <10ms between worker nodes
- <50ms for multi-site clusters

---

## Storage Options for Bare Metal

### 1. Local Storage

**Use case:** High-performance, node-local workloads

**Solutions:**
- Kubernetes local-path-provisioner
- Rancher Local Path Provisioner
- Direct PV mounts

**Pros:** Maximum performance
**Cons:** No redundancy, manual migration

---

### 2. Network Storage (NFS/Ceph)

**Use case:** Shared storage across nodes

**Solutions:**
- **NFS** - Simple, widely supported
- **Ceph** - Software-defined, highly available
- **Longhorn** - Kubernetes-native, Rancher-integrated

**Longhorn Example:**
```bash
# Install Longhorn via Rancher Catalog
# Or via Helm:
helm repo add longhorn https://charts.longhorn.io
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace
```

---

### 3. Distributed Block Storage

**Use case:** High availability, data replication

**Solutions:**
- **Longhorn** (recommended for Rancher)
- **OpenEBS**
- **Rook-Ceph**

---

## Provisioning Automation

### Metal as a Service (MAAS)

**What it is:** Canonical's bare metal provisioning system

**Features:**
- PXE boot automation
- OS image management
- API-driven provisioning
- IPMI/BMC integration

**Use case:** Large-scale bare metal fleet (10+ servers)

---

### Ironic (OpenStack)

**What it is:** OpenStack bare metal provisioning

**Use case:** If already using OpenStack infrastructure

---

### Manual Provisioning Checklist

For smaller deployments (3-10 servers):

- [ ] Install OS (Ubuntu/Rocky Linux)
- [ ] Configure static IPs
- [ ] Set up SSH key access
- [ ] Disable swap (`swapoff -a`)
- [ ] Configure firewall rules
- [ ] Install container runtime (containerd)
- [ ] Install RKE2/K3s
- [ ] Join to Rancher management

---

## Bare Metal Best Practices

### 1. Use Dedicated Control Plane Nodes

**Why:** Isolate control plane from workload fluctuations

```yaml
# Node tainting example
kubectl taint nodes control-plane-node-1 node-role.kubernetes.io/master:NoSchedule
```

---

### 2. Implement Node Labels

**Why:** Schedule workloads to appropriate hardware

```bash
# Label nodes by hardware type
kubectl label nodes gpu-node-1 hardware=gpu
kubectl label nodes storage-node-1 hardware=high-io

# Use node affinity in deployments
nodeSelector:
  hardware: gpu
```

---

### 3. Plan for Firmware Updates

**Strategy:**
- Schedule maintenance windows
- Use rolling updates (one node at a time)
- Test in staging first
- Keep spare capacity for node draining

---

### 4. Monitor Hardware Health

**Tools:**
- **Prometheus Node Exporter** - CPU, memory, disk metrics
- **IPMI Exporter** - Temperature, fan speed, power
- **SMART Monitoring** - Disk health

---

## Integration with Rancher

### Importing Existing Bare Metal Clusters

1. Deploy RKE2/K3s cluster on bare metal
2. In Rancher UI: **Cluster Management → Import Existing**
3. Run provided kubectl command on cluster
4. Cluster appears in Rancher dashboard

### Creating New Clusters via Rancher

**Requirements:**
- Node driver support (for automated provisioning)
- Or manual node registration

**Steps:**
1. Prepare bare metal nodes (OS installed, SSH access)
2. Rancher → Create → Custom Cluster
3. Add nodes via registration command
4. Rancher provisions Kubernetes

---

## Cost Comparison: Bare Metal vs Cloud

### 3-Year TCO Example (10-node cluster)

| Component | Bare Metal | AWS EC2 (m5.4xlarge) |
|-----------|------------|----------------------|
| **Hardware** | $80,000 (upfront) | - |
| **Compute** | - | $525,600 (3 years) |
| **Network** | $5,000/year | Included (egress extra) |
| **Storage** | $20,000 (upfront) | $36,000 (3 years) |
| **Power** | $15,000/year | - |
| **Total 3Y** | **$165,000** | **$561,600** |

**Break-even:** ~10 months for bare metal

**Note:** Cloud provides flexibility, bare metal provides cost savings at scale

---

## Banking-Specific Considerations

### Regulatory Compliance

- [ ] Physical access control (data center security)
- [ ] Audit trail for hardware changes
- [ ] Disposal procedures for decommissioned hardware
- [ ] Encryption at rest (LUKS for disk encryption)

### High Availability

- [ ] Dual power supplies
- [ ] Redundant network paths
- [ ] Geographic distribution (multi-site)
- [ ] Regular DR testing

### Security Hardening

- [ ] BIOS/UEFI password protection
- [ ] Secure Boot enabled
- [ ] TPM modules for encryption keys
- [ ] Physical port disabling (USB, etc.)

---

## Learning Path for Bare Metal

### Week 1: Setup Single Node
- Install Ubuntu on physical/virtual machine
- Deploy K3s
- Run sample workload
- Connect to Rancher (hosted trial)

### Week 2: Three-Node Cluster
- Provision 3 VMs/physical nodes
- Deploy RKE2 HA cluster
- Configure Longhorn storage
- Integrate with Rancher

### Week 3: Production Readiness
- Implement backup strategy
- Configure monitoring
- Test node failure scenarios
- Document runbooks

---

## Next Steps

- [ ] Define hardware procurement requirements
- [ ] Select OS distribution (Ubuntu vs Rocky Linux)
- [ ] Choose storage solution (Longhorn recommended)
- [ ] Plan network topology
- [ ] Create provisioning automation scripts

---

**Status:** Planning phase
**Last Updated:** January 2026
**Related:** [Cluster Topology](cluster-topology.md), [Hybrid Cloud](hybrid-cloud.md)
