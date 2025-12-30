# Contributing to Boilerplates CLI

Thank you for your interest in contributing to Boilerplates CLI! This document provides guidelines and instructions for contributing.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Making Contributions](#making-contributions)
- [Code Style](#code-style)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Template Contributions](#template-contributions)
- [Module Contributions](#module-contributions)

## Code of Conduct

Please be respectful and constructive in all interactions. We welcome contributors of all experience levels.

## Getting Started

### Prerequisites

- Python 3.9 or higher
- Git
- pipx (recommended) or pip

### Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/christianlempa/boilerplates.git
   cd boilerplates
   ```

2. **Create a virtual environment:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -e .
   pip install ruff yamllint pytest
   ```

4. **Verify installation:**
   ```bash
   python3 -m cli --version
   python3 -m cli compose list
   ```

## Making Contributions

### Types of Contributions

1. **Bug Fixes** - Fix existing issues
2. **New Templates** - Add Docker Compose, Terraform, etc. templates
3. **New Modules** - Add support for new infrastructure types
4. **Documentation** - Improve docs, examples, guides
5. **Tests** - Add or improve test coverage
6. **Features** - Implement new CLI features

### Workflow

1. **Check existing issues** - Look for related issues before creating new ones
2. **Create an issue** - Discuss significant changes before implementing
3. **Fork the repository** - Create your own fork
4. **Create a branch** - Use naming convention below
5. **Make changes** - Follow code style guidelines
6. **Test thoroughly** - Run linters and tests
7. **Submit PR** - Reference the related issue

### Branch Naming

```
feature/ISSUE-NUMBER-brief-description
problem/ISSUE-NUMBER-fix-description
docs/ISSUE-NUMBER-update-description
```

Examples:
- `feature/123-add-terraform-module`
- `problem/456-fix-variable-parsing`
- `docs/789-update-readme`

## Code Style

### Python

- Follow PEP 8
- Use 4-space indentation
- Type hints for public functions
- Google-style docstrings
- Maximum line length: 88 characters (ruff default)

```python
def process_template(
    template_id: str,
    output_dir: Path,
    variables: dict[str, Any],
) -> bool:
    """Process and render a template.

    Args:
        template_id: The unique template identifier.
        output_dir: Directory for rendered output.
        variables: Variable values for rendering.

    Returns:
        True if successful, False otherwise.

    Raises:
        TemplateNotFoundError: If template doesn't exist.
        TemplateRenderError: If rendering fails.
    """
    ...
```

### YAML

- Use 2-space indentation
- Maximum line length: 160 characters
- Use yamllint for validation

### Linting

Run before every commit:

```bash
# Python linting and formatting
ruff check cli/
ruff format cli/

# YAML linting
yamllint .
```

### Commit Messages

Format: `type(scope): subject`

Types:
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `refactor` - Code refactoring
- `test` - Adding tests
- `chore` - Maintenance tasks

Examples:
```
feat(compose): add postgresql template
fix(core): resolve variable validation edge case
docs(readme): add installation troubleshooting
refactor(template): simplify rendering pipeline
test(compose): add template generation tests
chore(deps): update typer to 0.20.0
```

## Testing

### Running Tests

```bash
# Template validation
./tests/test-compose-templates.sh

# Dry-run generation
python3 -m cli compose generate nginx --dry-run

# Debug mode
python3 -m cli --log-level DEBUG compose list
```

### Test Requirements

- All new templates must pass validation
- All new code should include appropriate tests
- Run full test suite before submitting PR

## Submitting Changes

### Pull Request Process

1. **Ensure all tests pass**
2. **Update documentation** if needed
3. **Add changelog entry** for notable changes
4. **Create PR with clear description:**
   - What does this PR do?
   - Related issue number
   - Testing performed
   - Screenshots (if UI changes)

### PR Title Format

Same as commit messages:
```
feat(compose): add redis template
fix(core): handle empty variable values
```

### PR Description Template

```markdown
## Summary
Brief description of changes.

## Related Issue
Fixes #123

## Changes Made
- Change 1
- Change 2

## Testing
- [ ] Ran linters (ruff, yamllint)
- [ ] Ran template validation
- [ ] Tested manually

## Checklist
- [ ] Code follows style guidelines
- [ ] Documentation updated
- [ ] Changelog updated (if applicable)
```

## Template Contributions

### Template Structure

```
library/compose/mytemplate/
├── template.yaml       # Required: metadata + variables
├── compose.yaml.j2     # Required: main template
├── config/             # Optional: config files
│   └── config.yaml.j2
└── data/               # Optional: static files
```

### template.yaml Requirements

```yaml
---
kind: compose
schema: "1.1"
metadata:
  name: My Service
  description: |
    Description with project links.

    Project: https://...
    Source: https://...
    Documentation: https://...
  version: 1.0.0
  author: Your Name
  date: '2025-01-01'
  tags: [category1, category2]
  draft: false  # Set true while developing
spec:
  general:
    title: General Settings
    required: true
    vars:
      container_name:
        type: str
        default: myservice
        description: Container name
      # Add more variables...
```

### Template Guidelines

1. **Use sensible defaults** - Templates should work out-of-box
2. **Document all variables** - Clear descriptions and examples
3. **Follow security best practices** - No hardcoded secrets
4. **Use standard naming** - Consistent variable names across templates
5. **Include comments** - Explain non-obvious configurations
6. **Test thoroughly** - Validate and dry-run before submitting

### Common Variables

Use these standard variables when applicable:

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `container_name` | str | service name | Docker container name |
| `container_timezone` | str | UTC | Container timezone |
| `restart_policy` | enum | unless-stopped | Restart policy |
| `traefik_enabled` | bool | false | Enable Traefik integration |
| `traefik_host` | hostname | - | Traefik hostname |

## Module Contributions

### Module Structure

For simple modules:
```
cli/modules/mymodule.py
```

For multi-schema modules:
```
cli/modules/mymodule/
├── __init__.py      # Module class
├── spec_v1_0.py     # Schema 1.0 spec
└── spec_v1_1.py     # Schema 1.1 spec
```

### Module Class

```python
from cli.core.module import Module
from cli.core.registry import registry
from cli.core.collection import VariableCollection


class MyModule(Module):
    """Module for managing MyTool configurations."""

    name = "mymodule"
    description = "Manage MyTool configurations"
    schema_version = "1.0"

    spec = VariableCollection.from_dict({
        "general": {
            "title": "General Settings",
            "required": True,
            "vars": {
                "my_var": {
                    "type": "str",
                    "default": "default_value",
                    "description": "Variable description",
                }
            }
        }
    })


# Register at module bottom
registry.register(MyModule)
```

### Module Guidelines

1. **Follow existing patterns** - Look at compose module for reference
2. **Use custom exceptions** - From `cli/core/exceptions.py`
3. **Use DisplayManager** - For all output
4. **Add semantic validators** - If applicable
5. **Support schema versioning** - For future compatibility
6. **Document thoroughly** - Update AGENTS.md with module info

## Getting Help

- **Questions:** Open a Discussion on GitHub
- **Bugs:** Open an Issue with `[Bug]` prefix
- **Features:** Open an Issue with `[Feature]` prefix
- **Discord:** Join Christian's Discord community

## Recognition

Contributors are recognized in:
- GitHub Contributors list
- Release notes for significant contributions
- CHANGELOG.md for notable changes

Thank you for contributing to Boilerplates CLI!

---

*Last updated: 2025-12-30*
