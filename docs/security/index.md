# Security Overview

## Banking Requirements

Our Kubernetes platform must satisfy:

- [ ] BNM compliance (Bank Negara Malaysia regulations)
- [ ] SOC 2 controls (if applicable)
- [ ] Internal security policies (InfoSec approval required)
- [ ] Audit trail for all actions
- [ ] Network segmentation per tenant
- [ ] Secret encryption at rest

## Topics

- **[RBAC & Authentication](rbac-auth.md)** - User access control
- **[Network Policies](network-policies.md)** - Traffic isolation
- **[Secrets Management](secrets.md)** - Credential storage
- **[Audit Logging](audit-logging.md)** - Compliance tracking

## Open Questions

- [ ] How does Rancher integrate with bank's Active Directory?
- [ ] What audit logs does Rancher provide by default?
- [ ] Can we enforce network policies via Rancher UI?
- [ ] Where are secrets stored (etcd encryption enabled)?
