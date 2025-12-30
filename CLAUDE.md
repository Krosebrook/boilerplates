# CLAUDE.md

> Comprehensive guidance for Claude AI when working with the Boilerplates CLI project.

## Project Identity

**Name:** Boilerplates CLI
**Type:** Python CLI for infrastructure template management
**Version:** 0.0.7
**Python:** 3.9+
**Author:** Christian Lempa
**License:** MIT

## Quick Context

This is a sophisticated CLI tool that:
1. Manages infrastructure templates (Docker Compose, Terraform, Ansible, Kubernetes, Packer)
2. Uses Jinja2 for templating with variable substitution
3. Supports multiple template libraries (git-based and local)
4. Provides interactive and non-interactive generation modes

## Architecture Overview

```
boilerplates/
├── cli/                    # Python CLI application
│   ├── __init__.py         # Version: "0.0.7"
│   ├── __main__.py         # Entry point, module discovery
│   ├── core/               # Core components
│   │   ├── collection.py   # VariableCollection dataclass
│   │   ├── config.py       # ConfigManager
│   │   ├── display.py      # DisplayManager (ALL output goes here)
│   │   ├── exceptions.py   # Custom exception hierarchy
│   │   ├── library.py      # LibraryManager
│   │   ├── module.py       # Abstract Module base class
│   │   ├── prompt.py       # PromptHandler (Rich library)
│   │   ├── registry.py     # Module auto-discovery
│   │   ├── repo.py         # Git repository management
│   │   ├── section.py      # VariableSection dataclass
│   │   ├── template.py     # Template parsing/rendering
│   │   ├── validators.py   # Semantic validators
│   │   ├── variable.py     # Variable dataclass
│   │   └── version.py      # Semantic versioning utils
│   └── modules/            # Technology-specific modules
│       └── compose/        # Docker Compose (only implemented module)
│           ├── __init__.py
│           ├── spec_v1_0.py
│           └── spec_v1_1.py
├── library/                # Template collections
│   └── compose/            # 33 Docker Compose templates
├── tests/                  # Test scripts
├── scripts/                # Installation scripts
└── docs/                   # Documentation
```

## Critical Patterns

### 1. Display Output
**NEVER** use `print()` or `console.print()` directly. Always use `DisplayManager`:

```python
from cli.core.display import display

# Correct
display.display_info("Processing template...")
display.display_success("Template generated!")
display.display_error("Template not found")
display.display_warning("Using default value")

# Wrong
print("Processing template...")  # NEVER DO THIS
console.print("[green]Done!")    # NEVER DO THIS
```

### 2. Exception Handling
**ALWAYS** use custom exceptions from `cli/core/exceptions.py`:

```python
from cli.core.exceptions import (
    TemplateNotFoundError,
    TemplateValidationError,
    VariableValidationError,
    ConfigError,
)

# Correct
raise TemplateNotFoundError(template_id="nginx", module_name="compose")

# Wrong
raise Exception("Template not found")  # NEVER DO THIS
```

### 3. Module Creation
Modules extend the abstract `Module` class:

```python
from cli.core.module import Module
from cli.core.registry import registry
from cli.core.collection import VariableCollection

class MyModule(Module):
    name = "mymodule"
    description = "Description for CLI help"
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

# Register at module bottom
registry.register(MyModule)
```

### 4. Template Structure
Each template is a directory:

```
template-name/
├── template.yaml       # Metadata + variable specs
├── compose.yaml.j2     # Jinja2 template (rendered)
├── config/             # Optional config files
│   └── file.conf.j2
└── data/               # Optional static files
```

### 5. Variable Precedence (lowest to highest)
1. Module spec defaults
2. Template spec overrides
3. User config.yaml
4. CLI `--var` arguments

## Common Tasks

### Adding a New Template

1. Create directory in `library/compose/mytemplate/`
2. Create `template.yaml`:
```yaml
---
kind: compose
schema: "1.1"
metadata:
  name: My Template
  description: |
    Description with links.

    Project: https://...
  version: 1.0.0
  author: Your Name
  date: '2025-01-01'
  tags: [monitoring, database]
spec:
  general:
    title: General Settings
    required: true
    vars:
      container_name:
        type: str
        default: myservice
        description: Container name
```
3. Create `compose.yaml.j2`:
```yaml
services:
  {{ container_name }}:
    image: myimage:{{ version }}
    restart: {{ restart_policy }}
```

### Adding a New Module

1. Create `cli/modules/mymodule/__init__.py`
2. Define module class with schema specs
3. Register with `registry.register(MyModule)`
4. CLI auto-discovers on startup

### Running Tests

```bash
# Run CLI
python3 -m cli compose list

# Debug mode
python3 -m cli --log-level DEBUG compose list

# Template validation
./tests/test-compose-templates.sh

# Dry-run generation
python3 -m cli compose generate nginx --dry-run
```

### Linting

```bash
# Python
ruff check cli/
ruff format cli/

# YAML
yamllint .
```

## Code Style

- **Python:** PEP 8, 4-space indentation
- **YAML:** 2-space indentation, 160 char max line
- **Docstrings:** Google style
- **Type hints:** Required for all public functions
- **Commits:** `type(scope): message` (e.g., `fix(compose): correct variable parsing`)

## Variable Types

| Type | Description | Validation |
|------|-------------|------------|
| `str` | String (default) | None |
| `int` | Integer | Number validation |
| `float` | Float | Number validation |
| `bool` | Boolean | True/False |
| `email` | Email address | Regex validation |
| `url` | URL | Scheme + host required |
| `hostname` | Hostname/domain | Domain validation |
| `enum` | Choice list | Must be in `options` |

## Variable Properties

```yaml
my_variable:
  type: str
  default: "default_value"
  description: "User-friendly description"
  prompt: "Custom prompt text"
  sensitive: true          # Masked in output
  autogenerated: true      # Auto-generate if empty
  required: true           # Always required
  optional: true           # Allow empty/None
  options: [a, b, c]       # For enum type
  extra: "Additional help"
```

## Section Properties

```yaml
my_section:
  title: "Display Title"
  required: true           # Always enabled
  toggle: bool_var_name    # Conditional on variable
  needs: [section1, sec2]  # Dependencies
  vars:
    # variables here
```

## Schema Versioning

- **Schema 1.0:** Basic functionality
- **Schema 1.1:** Adds `network_mode`, Docker Swarm support

Templates declare schema: `schema: "1.1"`
Modules support up to their `schema_version`

## Key Files to Know

| File | Purpose |
|------|---------|
| `cli/__init__.py` | Version string (`__version__`) |
| `pyproject.toml` | Package config, dependencies |
| `cli/core/exceptions.py` | All custom exceptions |
| `cli/core/display.py` | ALL output rendering |
| `cli/core/module.py` | Base class for modules |
| `cli/core/template.py` | Template parsing/rendering |

## WIP Features (Not Yet Implemented)

- Terraform module
- Docker (Dockerfile) module
- Ansible module
- Kubernetes module
- Packer module
- Wiki documentation
- PyPI publishing

## Don't Forget

1. **Update version in 3 places** for releases:
   - `cli/__init__.py`
   - `pyproject.toml`
   - `flake.nix`

2. **Run linters** before committing:
   - `ruff check cli/`
   - `yamllint .`

3. **Test templates** after changes:
   - `./tests/test-compose-templates.sh`

4. **Use DisplayManager** for ALL output

5. **Use custom exceptions** for ALL errors

## Helpful Commands

```bash
# Show all templates
python3 -m cli compose list

# Show template details
python3 -m cli compose show nginx

# Generate interactively
python3 -m cli compose generate nginx my-nginx

# Generate non-interactively
python3 -m cli compose generate nginx my-nginx \
  --var container_name=nginx \
  --no-interactive

# Validate all templates
python3 -m cli compose validate

# Repository operations
python3 -m cli repo list
python3 -m cli repo update
```

## Context Limits

When working with this codebase:
- Focus on the `cli/` directory for core logic
- The `library/compose/` has 33 template directories
- Each template is self-contained
- The codebase is ~7,600 lines of Python

## Questions to Ask

If requirements are unclear, ask about:
1. Which module (compose, terraform, etc.)?
2. Interactive or non-interactive mode?
3. Schema version requirements?
4. New template or module modification?
5. Breaking changes acceptable?

---

*This file helps Claude AI understand the Boilerplates CLI project structure, patterns, and conventions for more effective assistance.*
