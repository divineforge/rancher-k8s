# Rancher & Kubernetes Platform Research

Enterprise Kubernetes evaluation for banking environments.

## View Documentation

**Live Site:** https://divineforge.github.io/rancher-k8s/

## Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Run local server
mkdocs serve

# Open http://localhost:8000
```

## Deployment

Automatically deploys to GitHub Pages on push to `main` branch.

## Structure

- `docs/` - All markdown content
- `mkdocs.yml` - Site configuration
- `.github/workflows/` - Auto-deployment pipeline

## Documentation Sections

- **Getting Started** - Quick start guides for Rancher (hosted service, MacBook M1, bare metal)
- **Architecture** - Platform design, components, and bare metal management
- **Security** - Compliance and access control
- **Operations** - Monitoring, backup, day-2 ops
- **Developer Experience** - Onboarding and workflows
- **Decisions** - Why Rancher? Alternatives? Risks?
- **Progress** - Weekly research updates

## Contributing

This is a living document. Update as research progresses:

1. Edit markdown files in `docs/`
2. Preview locally with `mkdocs serve`
3. Commit and push to your branch
4. Create PR to `main` for review

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
