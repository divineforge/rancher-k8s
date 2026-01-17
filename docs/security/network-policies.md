# Network Policies

## Overview

Network segmentation and traffic control strategies for Kubernetes workloads.

## Policy Requirements

### Default Deny
- Deny all traffic by default
- Explicit allow rules for required communication
- Defense in depth approach

### Namespace Isolation
- Inter-namespace traffic restrictions
- Project-level network boundaries
- Egress control to external services

## Implementation

### CNI Plugin Options
- Calico (recommended for banking)
- Cilium
- Weave

### Policy Management
- GitOps for policy definitions
- Automated testing
- Policy audit trail

## Open Questions

- [ ] What CNI is currently deployed?
- [ ] Firewall integration requirements?
- [ ] Egress filtering to internet?

---

**Status:** Research phase
**Last Updated:** January 2026
