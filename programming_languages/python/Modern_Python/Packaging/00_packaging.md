# Packaging and Distribution

Learn to create distributable Python packages.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Package Structure**](01_structure.md) | Project layout |
| [**2. pyproject.toml**](02_pyproject.md) | Modern configuration |
| [**3. Publishing to PyPI**](03_publishing.md) | Distribution |

---

## Quick Reference

### Modern Package Structure
```
mypackage/
├── pyproject.toml       # Project metadata and build config
├── README.md
├── LICENSE
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── main.py
└── tests/
    └── test_main.py
```

### pyproject.toml
```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "0.1.0"
description = "A sample package"
readme = "README.md"
requires-python = ">=3.8"
license = {text = "MIT"}
authors = [
    {name = "Your Name", email = "you@example.com"}
]
dependencies = [
    "requests>=2.28",
]

[project.optional-dependencies]
dev = ["pytest", "black", "mypy"]

[project.scripts]
mycommand = "mypackage.main:main"
```

---

## Building and Publishing

```bash
# Install build tools
pip install build twine

# Build package
python -m build

# Upload to PyPI
twine upload dist/*

# Upload to Test PyPI
twine upload --repository testpypi dist/*
```

---

## Installation Modes

```bash
# Regular install
pip install mypackage

# Editable install (for development)
pip install -e .

# With optional dependencies
pip install -e ".[dev]"
```

---

## Next Steps
Start with [Package Structure](01_structure.md).
