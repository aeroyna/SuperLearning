# Best Practices and Idioms

Write clean, Pythonic, maintainable code.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. PEP 8 Style Guide**](01_pep8.md) | Formatting and naming |
| [**2. Pythonic Idioms**](02_pythonic_idioms.md) | Idiomatic Python |
| [**3. Common Pitfalls**](03_common_pitfalls.md) | Mistakes to avoid |

---

## Quick Reference

### Naming Conventions
```python
# Variables and functions: snake_case
user_name = "Alice"
def calculate_total():
    pass

# Classes: PascalCase
class UserAccount:
    pass

# Constants: UPPER_SNAKE_CASE
MAX_CONNECTIONS = 100

# Private: leading underscore
_internal_function()

# "Really private": double underscore
__very_private = 42
```

### Formatting
```python
# 4 spaces per indentation level
# 79 characters max line length (99 for modern projects)
# Two blank lines between top-level definitions
# One blank line between methods
```

---

## Pythonic Idioms

### Truthiness
```python
# Not Pythonic
if len(items) > 0:
if flag == True:

# Pythonic
if items:
if flag:
```

### Unpacking
```python
# Not Pythonic
first = items[0]
rest = items[1:]

# Pythonic
first, *rest = items
```

### Enumerate and Zip
```python
# Not Pythonic
for i in range(len(items)):
    print(i, items[i])

# Pythonic
for i, item in enumerate(items):
    print(i, item)

# Parallel iteration
for name, age in zip(names, ages):
    print(name, age)
```

### Context Managers
```python
# Not Pythonic
f = open("file.txt")
try:
    content = f.read()
finally:
    f.close()

# Pythonic
with open("file.txt") as f:
    content = f.read()
```

### Comprehensions
```python
# Not Pythonic
squares = []
for x in range(10):
    squares.append(x ** 2)

# Pythonic
squares = [x ** 2 for x in range(10)]
```

---

## Tools

```bash
# Formatting
pip install black
black mycode.py

# Linting
pip install flake8
flake8 mycode.py

# Import sorting
pip install isort
isort mycode.py

# Type checking
pip install mypy
mypy mycode.py
```

### pyproject.toml
```toml
[tool.black]
line-length = 88

[tool.isort]
profile = "black"

[tool.mypy]
strict = true
```

---

## The Zen of Python
```python
import this
```

Key principles:
- Beautiful is better than ugly
- Explicit is better than implicit
- Simple is better than complex
- Readability counts
- There should be one obvious way to do it

---

## Next Steps
Start with [PEP 8 Style Guide](01_pep8.md).
