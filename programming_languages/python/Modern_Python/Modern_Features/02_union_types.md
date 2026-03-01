# Union Types

Python 3.10 introduced a new, cleaner syntax for expressing Union types using the `|` operator (PEP 604).

## The Old Way (`Union`)
Before 3.10, you had to import `Union` from `typing`.

```python
from typing import Union, Optional

def parse(value: Union[int, str]) -> Union[float, None]:
    if isinstance(value, str):
        return float(value)
    return float(value)
```

## The New Way (`|`)
You can now use bitwise OR `|` on types. This is supported by `isinstance` and `issubclass` as well.

```python
def parse(value: int | str) -> float | None:
    if isinstance(value, str):
        return float(value)
    return float(value)
```

### `isinstance` checks
The syntax works at runtime too!

```python
# Old
isinstance(x, (int, str))

# New
isinstance(x, int | str)
```

## Optional Values
`Optional[T]` is just an alias for `Union[T, None]`.

```python
# Old
x: Optional[int] = None

# New
x: int | None = None
```

## Compatibility
This syntax requires Python 3.10+. If you need to support older versions (e.g., 3.8, 3.9) but want to use modern syntax, add:
```python
from __future__ import annotations
```
This allows the *syntax* in type hints (because evaluation is deferred), but `isinstance(x, int | str)` will fail at runtime on older versions.
