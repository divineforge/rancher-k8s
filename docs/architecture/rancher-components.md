# Rancher Components

## What is Rancher?

Rancher is a Kubernetes management platform that provides:
- Multi-cluster management
- Unified authentication/authorization
- Centralized monitoring and logging
- Catalog of pre-configured apps

## Core Components

### 1. Rancher Server
- Management control plane
- Hosts UI and API
- Stores cluster state
- **High Availability:** 3+ nodes recommended for production

### 2. Downstream Clusters
- Worker Kubernetes clusters managed by Rancher
- Run actual application workloads
- Independent of Rancher Server (survive Rancher outage)

### 3. Authentication Providers
- Active Directory/LDAP
- SAML (Okta, Azure AD)
- Local users (for testing)

## Architecture Diagram

```
┌─────────────────────────────────────┐
│      Rancher Server Cluster         │
│   (Management Plane - HA Setup)     │
└────────────┬────────────────────────┘
             │
    ┌────────┴────────┬─────────────┐
    │                 │             │
┌───▼──────┐  ┌───────▼────┐  ┌────▼─────┐
│  Dev K8s │  │  Staging   │  │   Prod   │
│  Cluster │  │  K8s       │  │   K8s    │
│          │  │  Cluster   │  │  Cluster │
└──────────┘  └────────────┘  └──────────┘
```

## Key Learnings

!!! note "Research Notes"
    - Rancher Server failure does NOT affect downstream workloads
    - Each downstream cluster needs Rancher agent installed
    - Agent communicates via websocket (firewall consideration)

## References

- [Rancher Architecture Guide](https://ranchermanager.docs.rancher.com/reference-guides/rancher-manager-architecture)

---

**Status:** Initial research
**Last Updated:** January 2026
**Next Steps:** [ ] Deploy test Rancher instance
