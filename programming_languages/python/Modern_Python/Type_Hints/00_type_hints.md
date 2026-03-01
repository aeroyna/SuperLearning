# Type Hints and Static Typing

Type hints enable static type checking while keeping Python's dynamic nature.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Basic Type Hints**](01_basic_type_hints.md) | Function and variable annotations |
| [**2. Advanced Types**](02_advanced_types.md) | Generics, protocols, TypedDict |
| [**3. Static Type Checkers**](03_static_checkers.md) | mypy, pyright |

---

## Quick Reference

### Basic Annotations
```python
# Variables
name: str = "Alice"
age: int = 30
active: bool = True
score: float = 98.5

# Functions
def greet(name: str) -> str:
    return f"Hello, {name}!"

def add(a: int, b: int) -> int:
    return a + b
```

### Common Types (typing module)
```python
from typing import List, Dict, Set, Tuple, Optional, Union

# Collections
names: List[str] = ["Alice", "Bob"]
scores: Dict[str, int] = {"Alice": 95, "Bob": 87}
unique: Set[int] = {1, 2, 3}
point: Tuple[int, int] = (10, 20)

# Optional (can be None)
result: Optional[str] = None  # str | None

# Union (multiple types)
value: Union[int, str] = 42  # int | str
```

### Python 3.10+ Syntax
```python
# Union with |
value: int | str = 42
result: str | None = None

# Built-in generics
names: list[str] = []
scores: dict[str, int] = {}
```

### Callable
```python
from typing import Callable

def apply(func: Callable[[int, int], int], a: int, b: int) -> int:
    return func(a, b)

# Function that takes (int, int) and returns int
```

---

## Type Aliases
```python
from typing import TypeAlias

# Simple alias
UserId: TypeAlias = int
UserDict: TypeAlias = dict[str, str]

# Complex types
Vector: TypeAlias = list[float]
Matrix: TypeAlias = list[Vector]
```

---

## Generics
```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self.items: list[T] = []

    def push(self, item: T) -> None:
        self.items.append(item)

    def pop(self) -> T:
        return self.items.pop()

stack: Stack[int] = Stack()
stack.push(1)
```

---

## Type Checking

```bash
# Install mypy
pip install mypy

# Check code
mypy script.py

# Strict mode
mypy --strict script.py
```

**Remember**: Type hints are NOT enforced at runtime — they're for static analysis.

---

## Next Steps
Start with [Basic Type Hints](01_basic_type_hints.md).
