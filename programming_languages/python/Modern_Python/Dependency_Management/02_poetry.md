# Poetry

Poetry is a modern tool for dependency management and packaging in Python. It solves the problems of `requirements.txt` and `setup.py` by unifying them.

## Key Features
*   **Dependency Resolution**: Ensures versions don't conflict.
*   **Lockfile (`poetry.lock`)**: Guarantees reproducible builds.
*   **Virtualenv Management**: Creates them automatically.
*   **Packaging**: Builds and publishes to PyPI.

## Basic Usage

```bash
# Initialize project
poetry init

# Add dependency
poetry add requests

# Add dev dependency
poetry add --group dev pytest

# Install dependencies (creates venv)
poetry install

# Run script in venv
poetry run python main.py
```

## The `pyproject.toml`
Poetry uses the standard configuration file.

```toml
[tool.poetry.dependencies]
python = "^3.10"
requests = "^2.28"
```
The `^` (caret) syntax means "Compatible with". `^2.28` means `>=2.28.0 <3.0.0`.
