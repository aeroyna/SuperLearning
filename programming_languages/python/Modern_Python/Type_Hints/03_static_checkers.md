# Static Checkers

Type hints in Python are forgotten at runtime (mostly). To get value from them, you must use a **static type checker**.

## Mypy

The reference implementation and most popular type checker.

### Installation
```bash
pip install mypy
```

### Usage
Run `mypy` against your file or directory.
```bash
mypy script.py
```

### Configuration (`mypy.ini`)
Strict config is recommended for new projects.

```ini
[mypy]
strict = True
disallow_untyped_defs = True
ignore_missing_imports = True
```

### Ignoring Errors
Sometimes mypy gets it wrong or you're using a dynamic library without stubs.

```python
import weird_lib

weird_lib.magic()  # type: ignore
```

## Other Tools

1.  **Pyright**: Microsoft's type checker (powers Pylance in VS Code). Extremely fast.
2.  **Pyre**: Meta's type checker. Optimized for massive monorepos.

## Typing Stubs (`.pyi`)
If a library doesn't include type hints, you can install separate stub files (e.g., `types-requests`) or write your own `.pyi` files to define the interface without implementation.

## Internals
Type checkers analyze the Abstract Syntax Tree (AST) and the Control Flow Graph (CFG) to prove type safety without executing code. They rely on "Type narrowing":

```python
def process(val: int | None):
    # val could be None here
    if val is not None:
        # Mypy knows val is strictly int here ("narrowed")
        print(val + 1)
```
