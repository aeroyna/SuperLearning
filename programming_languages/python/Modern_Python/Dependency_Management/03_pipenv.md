# Pipenv

Pipenv was an early attempt to bring the "Gemfile" (Ruby) or "package.json" (Node) workflow to Python. It combines `pip` and `virtualenv`.

## Key Features
*   `Pipfile`: Defines dependencies.
*   `Pipfile.lock`: Exact versions (hashes).
*   Auto-loads `.env` files.

## Basic Usage

```bash
# Install and create venv
pipenv install requests

# Activate shell
pipenv shell

# Run command
pipenv run python main.py
```

## Pipenv vs Poetry
While Pipenv is still widely used, **Poetry** is generally preferred for new projects today because:
1.  Poetry handles library packaging (publishing); Pipenv is just for applications.
2.  Poetry's dependency resolver is significantly faster.
3.  Poetry uses standard `pyproject.toml`; Pipenv uses custom `Pipfile`.
