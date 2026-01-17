# Getting Started with Rancher

## Overview

Multiple paths to start learning Rancher, depending on your resources and goals.

## Learning Paths

### 🌐 [Rancher Hosted Service](rancher-hosted.md)
**Best for:** Quick evaluation, no infrastructure needed

- **Time to start:** 5 minutes
- **Cost:** Free trial available
- **Learning focus:** Rancher UI, cluster import, app catalog
- **Limitations:** No full infrastructure control

---

### 💻 [MacBook M1 Pro Setup](macbook-m1.md)
**Best for:** Developers, local experimentation, learning Kubernetes

- **Time to start:** 30 minutes
- **Cost:** Free (uses local resources)
- **Learning focus:** K3s, Docker Desktop, Rancher Desktop, local development
- **Limitations:** Single-node cluster, resource constraints

---

### 🖥️ [Bare Metal Setup](bare-metal-setup.md)
**Best for:** Production evaluation, performance testing

- **Time to start:** 2-4 hours
- **Cost:** Requires physical/virtual servers
- **Learning focus:** Production architecture, HA setup, full stack
- **Limitations:** Infrastructure complexity

---

## Quick Comparison

| Feature | Hosted Service | MacBook M1 | Bare Metal |
|---------|----------------|------------|------------|
| **Setup Time** | 5 min | 30 min | 2-4 hours |
| **Cost** | Free trial | Free | Hardware required |
| **Multi-cluster** | ✅ Yes | ⚠️ Limited | ✅ Yes |
| **Production-like** | ⚠️ Partial | ❌ No | ✅ Yes |
| **Learning Curve** | Easy | Medium | Hard |
| **Best for** | UI exploration | Dev workflows | Production eval |

---

## Recommended Path

### Phase 1: Quick Start (Day 1)
Start with **[Rancher Hosted Service](rancher-hosted.md)** to:
- Explore the Rancher UI
- Import a demo cluster
- Deploy sample applications
- Understand multi-cluster management

### Phase 2: Local Development (Week 1)
Set up **[MacBook M1](macbook-m1.md)** to:
- Run Kubernetes locally
- Test deployments
- Learn kubectl commands
- Experiment without cloud costs

### Phase 3: Production Evaluation (Week 2+)
Deploy **[Bare Metal](bare-metal-setup.md)** to:
- Test HA configurations
- Validate performance
- Integrate with enterprise tools (AD, monitoring)
- Assess operational complexity

---

## Prerequisites

### All Paths
- Basic understanding of containers (Docker)
- Familiarity with command line
- GitHub account (for some tutorials)

### MacBook M1 Path
- macOS Monterey or newer
- 16GB RAM recommended (8GB minimum)
- 50GB free disk space
- Homebrew installed

### Bare Metal Path
- 3+ physical or virtual servers
- Ubuntu 22.04 or Rocky Linux 9
- Root/sudo access
- Basic networking knowledge

---

## Support Resources

- [Rancher Official Docs](https://ranchermanager.docs.rancher.com/)
- [Rancher Community Slack](https://slack.rancher.io/)
- [Rancher Forums](https://forums.rancher.com/)
- [GitHub Discussions](https://github.com/rancher/rancher/discussions)

---

## Next Steps

Choose your path and dive in:

- 🌐 **[Start with Rancher Hosted Service →](rancher-hosted.md)**
- 💻 **[Set up MacBook M1 →](macbook-m1.md)**
- 🖥️ **[Deploy Bare Metal →](bare-metal-setup.md)**

---

**Last Updated:** January 2026
