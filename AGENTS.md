# AGENTS.md

> Comprehensive guidance for AI agents and coding assistants working with this repository.

## Model-Specific Guidance

For AI-specific optimizations, see:
- **Claude:** [CLAUDE.md](./CLAUDE.md)
- **Gemini:** [GEMINI.md](./GEMINI.md)

---

## Project Overview

**Boilerplates CLI** is a sophisticated Python command-line tool for managing infrastructure templates. It provides instant access to production-ready templates for Docker Compose, Terraform, Ansible, Kubernetes, Packer, and more.

| Attribute | Value |
|-----------|-------|
| Version | 0.0.7 |
| Python | 3.9+ |
| Framework | Typer + Rich + Jinja2 |
| License | MIT |
| Author | Christian Lempa |

---

## Quick Start for Agents

```bash
# Run the CLI
python3 -m cli

# Debug mode
python3 -m cli --log-level DEBUG compose list

# List templates
python3 -m cli compose list

# Generate template
python3 -m cli compose generate nginx my-nginx

# Validate templates
python3 -m cli compose validate
```

### Linting (ALWAYS run before commits)

```bash
ruff check cli/     # Python
ruff format cli/    # Python formatting
yamllint .          # YAML
```

---

## Architecture

### Directory Structure

```
boilerplates/
├── cli/                          # Python CLI application
│   ├── __init__.py               # Version: "0.0.7"
│   ├── __main__.py               # Entry point, module discovery
│   ├── core/                     # Core framework
│   │   ├── collection.py         # VariableCollection dataclass
│   │   ├── config.py             # ConfigManager
│   │   ├── display.py            # DisplayManager (ALL output)
│   │   ├── exceptions.py         # Custom exceptions (ALWAYS use)
│   │   ├── library.py            # LibraryManager
│   │   ├── module.py             # Abstract Module base
│   │   ├── prompt.py             # PromptHandler (Rich)
│   │   ├── registry.py           # Module auto-discovery
│   │   ├── repo.py               # Git repository sync
│   │   ├── section.py            # VariableSection dataclass
│   │   ├── template.py           # Template parsing/rendering
│   │   ├── validators.py         # Semantic validators
│   │   ├── variable.py           # Variable dataclass
│   │   └── version.py            # Semver utilities
│   └── modules/                  # Technology modules
│       └── compose/              # Docker Compose (implemented)
│           ├── __init__.py       # ComposeModule
│           ├── spec_v1_0.py      # Schema 1.0
│           └── spec_v1_1.py      # Schema 1.1
├── library/                      # Template collections
│   └── compose/                  # 33 Docker Compose templates
├── tests/                        # Test scripts
├── scripts/                      # Installation scripts
└── docs/                         # Documentation
    └── ROADMAP.md                # Development roadmap
```

### Core Components

| Component | File | Purpose |
|-----------|------|---------|
| DisplayManager | `cli/core/display.py` | **ALL** CLI output (never print directly) |
| Exceptions | `cli/core/exceptions.py` | **ALL** errors (never use generic Exception) |
| Module | `cli/core/module.py` | Abstract base for technology modules |
| Template | `cli/core/template.py` | Template loading and rendering |
| VariableCollection | `cli/core/collection.py` | Variable sections and values |
| ConfigManager | `cli/core/config.py` | User configuration |
| LibraryManager | `cli/core/library.py` | Multi-library template discovery |
| PromptHandler | `cli/core/prompt.py` | Interactive prompts (Rich) |
| Registry | `cli/core/registry.py` | Module auto-discovery |

---

## Critical Patterns

### 1. Display Output (MANDATORY)

**ALWAYS** use DisplayManager for ALL output:

```python
from cli.core.display import display

# Correct
display.display_info("Processing...")
display.display_success("Done!")
display.display_error("Failed!")
display.display_warning("Caution!")

# WRONG - Never do this
print("message")           # NO
console.print("message")   # NO
```

### 2. Exception Handling (MANDATORY)

**ALWAYS** use custom exceptions:

```python
from cli.core.exceptions import (
    TemplateNotFoundError,
    VariableValidationError,
    ConfigError,
    TemplateRenderError,
)

# Correct
raise TemplateNotFoundError(template_id="nginx", module_name="compose")
raise VariableValidationError(variable_name="port", message="Must be 1-65535")

# WRONG - Never do this
raise Exception("Not found")     # NO
raise ValueError("Invalid")      # NO (unless type coercion)
```

### 3. Module Pattern

```python
from cli.core.module import Module
from cli.core.registry import registry
from cli.core.collection import VariableCollection

class MyModule(Module):
    name = "mymodule"
    description = "Module description for CLI help"
    schema_version = "1.0"

    spec = VariableCollection.from_dict({
        "general": {
            "title": "General Settings",
            "required": True,
            "vars": {
                "my_var": {
                    "type": "str",
                    "default": "value",
                    "description": "Help text"
                }
            }
        }
    })

# Register at bottom of module file
registry.register(MyModule)
```

---

## Templates

### Template Structure

```
library/compose/mytemplate/
├── template.yaml         # Metadata + variable specs (required)
├── compose.yaml.j2       # Main Jinja2 template (required)
├── config/               # Optional config files
│   └── settings.conf.j2
└── data/                 # Optional static files
```

### template.yaml Schema

```yaml
---
kind: compose                # Module type
schema: "1.1"                # Schema version (default: 1.0)
metadata:
  name: Service Name
  description: |
    Description with links.

    Project: https://...
    Source: https://...
    Documentation: https://...
  version: 1.0.0
  author: Author Name
  date: '2025-01-01'
  tags: [monitoring, database]
  draft: false               # Set true for WIP templates
  next_steps: |              # Optional post-generation instructions
spec:
  section_name:
    title: Display Title
    required: true/false
    toggle: bool_var_name    # Conditional section
    needs: [section1, sec2]  # Dependencies
    vars:
      variable_name:
        type: str/int/float/bool/email/url/hostname/enum
        default: value
        description: Help text
        prompt: Custom prompt text
        sensitive: true/false
        autogenerated: true/false
        required: true/false
        optional: true/false
        options: [a, b, c]   # For enum type
        extra: Additional help
```

### Variable Types

| Type | Description | Validation |
|------|-------------|------------|
| `str` | String (default) | None |
| `int` | Integer | Numeric |
| `float` | Float | Numeric |
| `bool` | Boolean | true/false |
| `email` | Email address | Regex |
| `url` | URL | Scheme required |
| `hostname` | Domain/hostname | Domain format |
| `enum` | Choice | Must be in `options` |

### Variable Precedence (lowest to highest)

1. Module `spec` defaults
2. Template `spec` overrides
3. User `config.yaml`
4. CLI `--var` arguments

### Schema Versioning

- **Schema 1.0:** Basic functionality
- **Schema 1.1:** `network_mode`, Docker Swarm support

Templates declare: `schema: "1.1"`
Modules support up to their `schema_version`

---

## Commands

### Standard Module Commands (auto-registered)

| Command | Description |
|---------|-------------|
| `list` | List all templates |
| `search <query>` | Search templates by ID |
| `show <id>` | Show template details |
| `generate <id> [dir]` | Generate from template |
| `validate [id]` | Validate templates |
| `defaults` | Manage user defaults |

### Generate Options

```bash
--dry-run           # Preview without writing
--no-interactive    # Skip prompts, use defaults
--var KEY=VALUE     # Override variable
```

### Core Commands

```bash
boilerplates repo list      # List configured libraries
boilerplates repo update    # Sync git libraries
boilerplates repo add       # Add new library
boilerplates repo remove    # Remove library
```

---

## LibraryManager

### Library Types

1. **git:** Synced from remote repository
2. **static:** Local directory

### Configuration

```yaml
# ~/.config/boilerplates/config.yaml
libraries:
  - name: default
    type: git
    url: https://github.com/user/templates.git
    branch: main
    directory: library
  - name: local
    type: static
    path: ~/my-templates
```

### Template Resolution

- Priority: Config order (first = highest)
- Simple ID: First library match wins
- Qualified ID: `template.library` for specific library

---

## Validation

### Jinja2 Validation

- Syntax error detection
- Undefined variable detection
- Built into Template class

### Semantic Validation

- `DockerComposeValidator`: Docker Compose schema
- `YAMLValidator`: YAML structure
- Extensible via `ContentValidator` base class

---

## Development Guidelines

### Git Workflow

```
feature/ISSUE-brief-description
problem/ISSUE-fix-description
```

### Commit Messages

```
type(scope): subject

feat(compose): add new template
fix(core): resolve variable parsing
docs(readme): update installation
refactor(template): simplify rendering
test(compose): add validation tests
chore(deps): update dependencies
```

### Code Style

- **Python:** PEP 8, 4-space indent, type hints
- **YAML:** 2-space indent, 160 char max
- **Docstrings:** Google style

### Version Sync

Update version in 3 places:
1. `cli/__init__.py`
2. `pyproject.toml`
3. `flake.nix`

---

## Current Status

### Implemented

- [x] Core CLI framework
- [x] Docker Compose module (33 templates)
- [x] Multi-library support
- [x] Schema versioning (1.0, 1.1)
- [x] Interactive/non-interactive modes
- [x] Semantic validation
- [x] Rich terminal UI

### Work in Progress

- [ ] Terraform module
- [ ] Docker (Dockerfile) module
- [ ] Ansible module
- [ ] Kubernetes module
- [ ] Packer module
- [ ] Wiki documentation
- [ ] PyPI publishing

---

## Testing

```bash
# Template validation
./tests/test-compose-templates.sh

# Single template
python3 -m cli compose validate nginx

# Dry-run
python3 -m cli compose generate nginx --dry-run

# Debug mode
python3 -m cli --log-level DEBUG compose list
```

---

## Quick Reference

### Key Files

| Purpose | Location |
|---------|----------|
| Version | `cli/__init__.py` |
| Dependencies | `pyproject.toml` |
| All output | `cli/core/display.py` |
| All errors | `cli/core/exceptions.py` |
| Module base | `cli/core/module.py` |
| Templates | `library/compose/` |

### Common Imports

```python
from cli.core.display import display
from cli.core.exceptions import TemplateNotFoundError, VariableValidationError
from cli.core.module import Module
from cli.core.registry import registry
from cli.core.collection import VariableCollection
from cli.core.template import Template
from cli.core.config import ConfigManager
```

---

*For detailed development roadmap, see [docs/ROADMAP.md](./docs/ROADMAP.md)*
*For contribution guidelines, see [CONTRIBUTING.md](./CONTRIBUTING.md)*
