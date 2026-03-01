# Dependency Management

Modern tools for managing Python dependencies.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. pip**](01_pip.md) | Standard package installer |
| [**2. Poetry**](02_poetry.md) | Dependency management and packaging |
| [**3. pipenv**](03_pipenv.md) | Virtual environments + pip |

---

## Quick Comparison

| Tool | Lock File | Virtual Env | Publishing |
|------|-----------|-------------|------------|
| pip | requirements.txt (manual) | Separate | Separate |
| Poetry | poetry.lock | Built-in | Built-in |
| pipenv | Pipfile.lock | Built-in | No |

---

## pip
```bash
pip install package
pip install -r requirements.txt
pip freeze > requirements.txt
```

## Poetry
```bash
# Install
curl -sSL https://install.python-poetry.org | python3 -

# Create project
poetry new myproject

# Add dependencies
poetry add requests
poetry add --group dev pytest

# Install dependencies
poetry install

# Run commands
poetry run python script.py
poetry run pytest

# Build and publish
poetry build
poetry publish
```

### pyproject.toml (Poetry)
```toml
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""
authors = ["You <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.10"
requests = "^2.28"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0"
black = "^22.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

---

## pipenv
```bash
# Install
pip install pipenv

# Create/activate environment
pipenv shell

# Add dependencies
pipenv install requests
pipenv install --dev pytest

# Install from Pipfile
pipenv install

# Run commands
pipenv run python script.py
```

---

## Recommendation

- **Small projects**: pip + venv + requirements.txt
- **Libraries/packages**: Poetry
- **Applications**: Poetry or pipenv

---

## Next Steps
Start with [pip](01_pip.md).
