# Week 1: Research Phase

## Objectives

- [x] Understand Rancher architecture
- [x] Map security requirements
- [ ] Interview stakeholders
- [ ] Document alternatives

## Key Learnings

### Architecture

- Rancher uses **management cluster + downstream clusters** model
- Downstream clusters survive Rancher outage (good for resilience)
- Agent-based communication (need to check firewall rules)

### Security

- Built-in RBAC templates
- AD/LDAP integration via standard protocols
- Audit logs available (need to validate format for compliance)

### Open Questions

!!! warning "Blockers"
    - [ ] Do we have VM quota for 6+ nodes (3 Rancher + 3 downstream)?
    - [ ] Who owns firewall approval for cluster communication?
    - [ ] What's InfoSec timeline for security review?

## Stakeholder Meetings

### InfoSec Team (Pending)
**Questions to ask:**
- What audit log format do you require?
- Can we use existing AD integration or need new setup?
- Network segmentation requirements?

### App Team Alpha (Pending)
**Questions to ask:**
- Current deployment pain points?
- Would self-service K8s help?
- What's your CI/CD stack?

## Documentation Created

- Architecture overview
- Security requirements mapping
- Decision rationale
- Alternatives comparison

## Next Week Focus

- Secure infrastructure for PoC
- Deploy test Rancher instance
- Document hands-on learnings
- Complete stakeholder interviews

---

**Week 1 Status:** 60% complete
**Blockers:** Infrastructure access pending
**Last Updated:** January 2026
