# Project Structure

Organizing your Python project correctly is the first step towards maintainability and easy packaging.

## The `src` Layout (Recommended)

The modern standard is the "src layout", where package code lives in a `src/` subdirectory.

```
my-project/
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── module.py
├── tests/
├── pyproject.toml
└── README.md
```

### Why `src`?
1.  **Prevents "import mismatch"**: You can't accidentally import the local folder when running tests; you *must* install the package (in editable mode) to test it. This ensures you are testing what you will actually ship.
2.  **Cleaner root**: Keeps the root directory for config files.

## The Flat Layout (Legacy/Simple)

Code sits directly in the root.

```
my-project/
├── my_package/
│   ├── __init__.py
│   └── module.py
├── tests/
├── pyproject.toml
└── README.md
```
Acceptable for simple scripts or dockerized apps where packaging isn't the primary goal, but `src` is preferred for libraries.

## Vital Files
*   `__init__.py`: Marks directory as a package. Can be empty.
*   `pyproject.toml`: The build configuration (see next chapter).
*   `README.md`: Documentation.
*   `.gitignore`: Git exclusions (`__pycache__`, `*.pyc`, `venv/`).
