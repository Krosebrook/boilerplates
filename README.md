# Boilerplates CLI

[![GitHub Release](https://img.shields.io/github/v/release/christianlempa/boilerplates?style=flat-square)](https://github.com/christianlempa/boilerplates/releases)
[![Python Version](https://img.shields.io/badge/python-3.9%2B-blue?style=flat-square)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

[![Welcome](https://cnd-prod-1.s3.us-west-004.backblazeb2.com/new-banner4-scaled-for-github.jpg)](https://youtu.be/apgp9egIKK8)

**Hey, there!**

**I'm Christian, and I'm passionate about creating educational tech content for IT Pros and Homelab nerds.**

## What are Boilerplates?

**Boilerplates** is a curated collection of production-ready templates for your homelab and infrastructure projects. Stop copying configurations from random GitHub repos or starting from scratch every time you spin up a new service!

## Boilerplates CLI

The Boilerplates CLI tool gives you instant access to battle-tested templates for Docker, Terraform, Ansible, Kubernetes, and more.

Each template includes sensible defaults, best practices, and common configuration patterns - so you can focus on customizing for your environment.

### Key Features

| Feature | Description |
|---------|-------------|
| **Quick Setup** | Generate complete project structures in seconds |
| **Customizable** | Interactive prompts or non-interactive mode with variable overrides |
| **Smart Defaults** | Save your preferred values and reuse across projects |
| **Multi-Library** | Use multiple template sources (git repos + local directories) |
| **Schema Versioning** | Backward-compatible template evolution |
| **Validation** | Built-in Jinja2 and semantic validation |

> **Note:** Technologies evolve rapidly. While I actively maintain these templates, always review generated configurations before deploying to production.

---

## Installation

### Automated Installer (Recommended)

```bash
# Install latest version
curl -fsSL https://raw.githubusercontent.com/christianlempa/boilerplates/main/scripts/install.sh | bash

# Install specific version
curl -fsSL https://raw.githubusercontent.com/christianlempa/boilerplates/main/scripts/install.sh | bash -s -- --version v0.0.7
```

The installer uses `pipx` to create an isolated environment. Once installed, the `boilerplates` command will be available in your terminal.

### NixOS / Nix Flakes

```bash
# Run without installing
nix run github:christianlempa/boilerplates -- --help

# Install to your profile
nix profile install github:christianlempa/boilerplates

# Use in a temporary shell
nix shell github:christianlempa/boilerplates

# Add to your flake
{
  inputs.boilerplates.url = "github:christianlempa/boilerplates";
  outputs = { self, nixpkgs, boilerplates }: {
    # Use boilerplates.packages.${system}.default
  };
}
```

### Manual Installation

```bash
# Clone and install with pip
git clone https://github.com/christianlempa/boilerplates.git
cd boilerplates
pip install -e .
```

---

## Quick Start

```bash
# Explore available commands
boilerplates --help

# Update template library
boilerplates repo update

# List all Docker Compose templates
boilerplates compose list

# Show template details
boilerplates compose show nginx

# Generate a template (interactive mode)
boilerplates compose generate authentik

# Generate with custom output directory
boilerplates compose generate nginx my-nginx-server

# Non-interactive mode with variable overrides
boilerplates compose generate traefik my-proxy \
  --var service_name=traefik \
  --var traefik_enabled=true \
  --var traefik_host=proxy.example.com \
  --no-interactive

# Dry-run (preview without writing files)
boilerplates compose generate nginx --dry-run

# Validate templates
boilerplates compose validate
```

---

## Available Templates

### Docker Compose (33 templates)

| Category | Templates |
|----------|-----------|
| **Monitoring** | Prometheus, Grafana, Loki, Alloy, Wazuh, Checkmk, Uptime Kuma |
| **Dashboards** | Heimdall, Homer, Homepage, Portainer, Dockge, Nginx Proxy Manager |
| **DevOps** | Gitea, GitLab, GitLab Runner, SemaphoreUI |
| **Databases** | PostgreSQL, MariaDB, InfluxDB |
| **Auth** | Authentik, Passbolt |
| **Infrastructure** | Nginx, Traefik, Bind9, Pi-hole, Twingate |
| **Content** | Nextcloud, n8n, Home Assistant, Open WebUI |
| **Utilities** | Whoami, ClamAV |

### Coming Soon

- Terraform/OpenTofu templates
- Docker (Dockerfile) templates
- Ansible playbooks
- Kubernetes manifests
- Packer templates

---

## Managing Defaults

Save time by setting default values for frequently used variables:

```bash
# Set default values
boilerplates compose defaults set container_timezone "America/New_York"
boilerplates compose defaults set restart_policy "unless-stopped"

# List current defaults
boilerplates compose defaults list

# Remove a default
boilerplates compose defaults rm container_timezone
```

---

## Template Libraries

Boilerplates uses git-based libraries to manage templates. You can add custom repositories:

```bash
# List configured libraries
boilerplates repo list

# Update all libraries
boilerplates repo update

# Add a custom library
boilerplates repo add my-templates https://github.com/user/templates \
  --directory library \
  --branch main

# Remove a library
boilerplates repo remove my-templates
```

### Library Priority

When multiple libraries contain the same template ID:
- Libraries are checked in config order (first = highest priority)
- Use qualified IDs for specific libraries: `nginx.my-templates`

---

## Configuration

Configuration is stored in `~/.config/boilerplates/config.yaml`:

```yaml
libraries:
  - name: default
    type: git
    url: https://github.com/christianlempa/boilerplates.git
    branch: main
    directory: library
  - name: local
    type: static
    path: ~/my-templates

defaults:
  compose:
    container_timezone: "America/New_York"
    restart_policy: "unless-stopped"
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [ROADMAP](docs/ROADMAP.md) | Development roadmap to post-MVP |
| [CONTRIBUTING](CONTRIBUTING.md) | Contribution guidelines |
| [AGENTS](AGENTS.md) | AI/coding assistant guidance |
| [CHANGELOG](CHANGELOG.md) | Version history |
| [SECURITY](SECURITY.md) | Security policy |

### For AI Assistants

- [CLAUDE.md](CLAUDE.md) - Claude AI guidance
- [GEMINI.md](GEMINI.md) - Gemini AI guidance
- [AGENTS.md](AGENTS.md) - General AI agent guidance

---

## Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Quick Contribution Guide

1. Fork the repository
2. Create a feature branch: `feature/your-feature`
3. Make changes and run linters: `ruff check cli/ && yamllint .`
4. Submit a pull request

---

## Other Resources

- [Dotfiles](https://github.com/christianlempa/dotfiles) - My personal configuration files on macOS
- [Cheat-Sheets](https://github.com/christianlempa/cheat-sheets) - Command Reference for various tools and technologies
- [YouTube Channel](https://www.youtube.com/@christianlempa) - Tutorials and educational content

---

## Support

Creating high-quality videos and valuable resources that are accessible to everyone, free of charge, is a huge challenge. With your contribution, I can dedicate more time and effort into the creation process, which ultimately enhances the quality of the content.

All your support truly makes a significant impact. And you'll also get some cool benefits and perks in return!

Remember, **supporting me is entirely optional.** Your choice to become a member won't change your access to my videos and resources.

[https://www.patreon.com/christianlempa](https://www.patreon.com/christianlempa)

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

*Made with love by [Christian Lempa](https://github.com/christianlempa)*
