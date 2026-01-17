# Self-Service Model

## Overview

Enable developers to provision resources and deploy applications without manual ops intervention.

## Self-Service Capabilities

### Namespace Provisioning
- Automated namespace creation
- Default resource quotas
- Network policies applied
- RBAC configured

### Application Deployment
- Helm chart catalog
- Pre-approved application templates
- Database provisioning
- Ingress configuration

### Resource Management
- Scale deployments
- View logs and metrics
- Restart pods
- Update configurations

## Guardrails

### Resource Limits
- CPU/memory quotas per namespace
- Storage limits
- Pod count limits

### Security Policies
- Image pull policies
- Security contexts
- Network egress rules

### Approval Workflows
- Production deployments require approval
- External service integrations reviewed
- Database provisioning gated

## Implementation

### Rancher Projects
- Project = collection of namespaces
- Team-based isolation
- Shared quotas and policies

### Service Catalog
- Curated Helm charts
- Internal application templates
- Database operators

### Automation
- GitOps for configuration
- Webhooks for automation
- Slack/Teams integration

## Future Enhancements

- [ ] Internal developer portal
- [ ] Cost visibility per team
- [ ] Automated testing environments
- [ ] Preview environments for PRs

---

**Status:** Planning phase
**Last Updated:** January 2026
