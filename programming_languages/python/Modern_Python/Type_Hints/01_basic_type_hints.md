# Basic Type Hints

Python 3.5 introduced type hints (PEP 484), allowing for optional static typing. This improves code readability and enables catch bugs early via static analysis tools.

## Basics

### Variables
```python
x: int = 5
name: str = "Alice"
active: bool = True
```

### Functions
Specify argument types and return types using `->`.

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

def add(x: int, y: int) -> int:
    return x + y
```

### Collections
For Python 3.9+, use built-in collection types. For older versions, import from `typing`.

```python
# Python 3.9+
names: list[str] = ["Alice", "Bob"]
scores: dict[str, int] = {"Alice": 100, "Bob": 90}
coords: tuple[int, int] = (10, 20)

# Pre-3.9 (Classic)
from typing import List, Dict, Tuple
names: List[str] = ["Alice", "Bob"]
```

---

## Special Types

### `Optional` / `None`
If a value can be `None`, it must be declared.
```python
from typing import Optional

def find_user(id: int) -> Optional[str]:
    # Returns str OR None
    return "User"
```
Python 3.10+ syntax:
```python
def find_user(id: int) -> str | None:
    return "User"
```

### `Union`
When a variable can be one of several types.
```python
from typing import Union

x: Union[int, str] = 5
x = "hello"  # Also valid
```
Python 3.10+ syntax:
```python
x: int | str = 5
```

### `Any`
The "escape hatch". Disables type checking for this variable.
```python
from typing import Any

def explicit(x: Any) -> None:
    print(x.undefined_method())  # Mypy won't complain!
```
Use `Any` sparingly!

---

## Best Practices
1.  **Annotate function boundaries**: Always type hint arguments and return values.
2.  **Avoid `Any`**: It defeats the purpose of type hints.
3.  **Use `|` syntax**: If on Python 3.10+, prefer `int | str` over `Union[int, str]`.
