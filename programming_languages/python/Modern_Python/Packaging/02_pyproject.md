# pyproject.toml

Introduced by PEP 518, `pyproject.toml` is the new standard for Python project configuration, replacing `setup.py`, `setup.cfg`, `requirements.txt`, and tool-specific configs (like `pytest.ini`).

## Build System
Defines what tool builds your package (e.g., setuptools, flit, poetry).

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

## Project Metadata (PEP 621)
Standardized metadata keys.

```toml
[project]
name = "my-awesome-package"
version = "0.1.0"
authors = [
  { name="Jane Doe", email="jane@example.com" },
]
description = "A package that does everything"
readme = "README.md"
requires-python = ">=3.9"
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]
dependencies = [
    "requests>=2.28",
    "numpy",
]

[project.optional-dependencies]
dev = ["pytest", "black"]
```

## Tool Configuration
Centralize config for linting, testing, etc.

```toml
[tool.pytest.ini_options]
addopts = "-ra -q"
testpaths = ["tests"]

[tool.black]
line-length = 88
```

## Setup.py?
For pure Python projects, `setup.py` is largely obsolete. Use `pyproject.toml` unless you need complex C-extension compilation logic.
