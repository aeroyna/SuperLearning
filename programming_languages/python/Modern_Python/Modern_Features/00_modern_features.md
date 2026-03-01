# Python 3.10+ Features

New features in modern Python versions.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Pattern Matching**](01_pattern_matching.md) | Structural pattern matching (3.10) |
| [**2. Union Types**](02_union_types.md) | X \| Y syntax (3.10) |
| [**3. Python 3.11-3.12**](03_latest_features.md) | Recent additions |

---

## Python 3.10

### Pattern Matching
```python
match command.split():
    case ["quit"]:
        quit()
    case ["load", filename]:
        load_file(filename)
    case ["save", filename]:
        save_file(filename)
    case _:
        print("Unknown command")
```

### Union Type Operator
```python
# Old
from typing import Union
def func(x: Union[int, str]) -> Union[int, str]:
    ...

# New (3.10+)
def func(x: int | str) -> int | str:
    ...

# Optional
def func(x: str | None = None):
    ...
```

### Better Error Messages
```python
# More helpful syntax error messages
# Points directly to the issue
```

---

## Python 3.11

### Exception Groups
```python
try:
    ...
except* ValueError as eg:
    print(f"Value errors: {eg.exceptions}")
except* TypeError as eg:
    print(f"Type errors: {eg.exceptions}")
```

### Performance (10-60% faster)
- Faster startup
- Faster function calls
- Specialized bytecode

### Tomllib
```python
import tomllib

with open("config.toml", "rb") as f:
    config = tomllib.load(f)
```

---

## Python 3.12

### Per-Interpreter GIL
Multiple interpreters with separate GILs for better parallelism.

### Type Parameter Syntax
```python
# Old
from typing import TypeVar
T = TypeVar('T')
def first(items: list[T]) -> T: ...

# New (3.12+)
def first[T](items: list[T]) -> T: ...

class Stack[T]:
    def push(self, item: T) -> None: ...
```

### Better f-strings
```python
# Nested quotes allowed
f"Hello {person['name']}"
```

---

## Feature Timeline

| Version | Key Features |
|---------|-------------|
| 3.8 | Walrus operator, positional-only params |
| 3.9 | Dict union operators, type hint generics |
| 3.10 | Pattern matching, union type syntax |
| 3.11 | Exception groups, toml, speed improvements |
| 3.12 | Type param syntax, per-interpreter GIL |

---

## Next Steps
Start with [Pattern Matching](01_pattern_matching.md).
