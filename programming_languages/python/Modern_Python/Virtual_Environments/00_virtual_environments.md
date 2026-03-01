# Virtual Environments

Virtual environments isolate project dependencies, preventing conflicts between projects.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Creating Environments**](01_creating.md) | venv, virtualenv |
| [**2. Managing Dependencies**](02_dependencies.md) | pip, requirements.txt |

---

## Quick Reference

### venv (Built-in)
```bash
# Create
python -m venv .venv

# Activate (Linux/Mac)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Deactivate
deactivate
```

### virtualenv
```bash
# Install
pip install virtualenv

# Create
virtualenv .venv

# With specific Python version
virtualenv -p python3.11 .venv
```

### pip Operations
```bash
# Install package
pip install requests

# Install specific version
pip install requests==2.28.0

# Install from requirements
pip install -r requirements.txt

# Generate requirements
pip freeze > requirements.txt

# Show installed packages
pip list

# Show package info
pip show requests
```

---

## Best Practices

1. **Always use virtual environments** for projects
2. **Name consistently** — `.venv` or `venv`
3. **Add to .gitignore** — don't commit the environment
4. **Keep requirements.txt** up to date
5. **Pin versions** in production

### .gitignore
```
.venv/
venv/
__pycache__/
*.pyc
```

---

## Project Structure
```
myproject/
├── .venv/              # Virtual environment (not in git)
├── src/
│   └── mypackage/
├── tests/
├── requirements.txt    # Production dependencies
├── requirements-dev.txt  # Dev dependencies
└── pyproject.toml
```

---

## Next Steps
Start with [Creating Environments](01_creating.md).
