# Why Rancher?

## Decision Summary

!!! success "Recommendation"
    **Proceed with Rancher evaluation** based on initial research.

    **Confidence Level:** Medium (need PoC validation)

## Problems We're Solving

1. **Multi-cluster chaos** - Teams deploying K8s clusters without governance
2. **Authentication complexity** - Each cluster has separate user management
3. **No visibility** - Can't see what's running where
4. **Security gaps** - Inconsistent network policies, RBAC models
5. **Developer friction** - Each team reinvents deployment pipelines

## Why Rancher (Hypothesis)

### ✅ Pros

**1. Centralized Management**
- Single pane of glass for all clusters
- Unified RBAC across environments
- Consistent policy enforcement

**2. Enterprise Features**
- Active Directory integration (bank requirement)
- Audit logging built-in
- Multi-tenancy support

**3. Developer Experience**
- Catalog of pre-approved apps (Helm charts)
- GitOps via Rancher Fleet
- Self-service namespace provisioning

**4. Hybrid Cloud Ready**
- Manages on-prem + cloud clusters equally
- No vendor lock-in to specific cloud

**5. Open Source Foundation**
- Can switch to community edition if SUSE relationship fails
- Large community support

### ⚠️ Cons / Risks

**1. Vendor Dependency**
- Reliance on SUSE for support
- Licensing costs (Prime edition for enterprise features)

**2. Complexity Layer**
- Another moving part to maintain
- Team needs to learn Rancher + Kubernetes

**3. Performance Overhead**
- Management plane resource consumption
- Network latency for agent communication

**4. Migration Risk**
- If Rancher fails, can we migrate to native K8s tools?
- What's the exit strategy?

## Alternatives Considered

| Platform | Pros | Cons | Verdict |
|----------|------|------|---------|
| **OpenShift** | Enterprise support, banking track record | Expensive, Red Hat lock-in | ❌ Cost |
| **Tanzu** | VMware integration | Complex, expensive | ❌ Cost |
| **DIY (native K8s)** | Full control, no licensing | High operational burden | ⚠️ Backup option |

## Next Steps

- [ ] Week 2: Deploy Rancher PoC
- [ ] Week 3: Test AD integration
- [ ] Week 4: InfoSec review

---

**Decision Date:** January 2026
**Revisit Date:** End of Week 2 (after PoC)
