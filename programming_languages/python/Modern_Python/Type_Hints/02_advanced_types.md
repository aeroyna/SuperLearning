# Advanced Type Hints

Beyond basic primitives, Python's typing system supports powerful constructs for complex data structures and patterns.

## Generics (`TypeVar`)

Used when functions or classes work with multiple types but need to maintain a relationship between them (e.g., return type matches argument type).

```python
from typing import TypeVar, list

T = TypeVar('T')

def get_first(items: list[T]) -> T:
    return items[0]

# Mypy infers T is int
x = get_first([1, 2, 3]) 

# Mypy infers T is str
y = get_first(["a", "b"])
```

## `Callable`
Typing functions passed as arguments.

```python
from typing import Callable

# Function taking two ints and returning an int
Operation = Callable[[int, int], int]

def apply(op: Operation, x: int, y: int) -> int:
    return op(x, y)
```

## `NewType`
Creates distinct types for values that are otherwise identical at runtime. useful for preventing logic errors (e.g., confusing UserID with ProductID).

```python
from typing import NewType

UserId = NewType('UserId', int)
ProductId = NewType('ProductId', int)

def get_user(uid: UserId):
    pass

u = UserId(12345)
p = ProductId(12345)

get_user(u)  # OK
# get_user(p)  # Error: Expected UserId, got ProductId
```

## `Literal`
Restricts a value to a specific set of literal values (enums without defining an Enum).

```python
from typing import Literal

Mode = Literal['r', 'w', 'rb', 'wb']

def open_file(file: str, mode: Mode):
    pass

open_file("test.txt", "r")   # OK
# open_file("test.txt", "x")   # Error
```

## `TypedDict`
For dictionaries with a fixed set of keys and value types (similar to a strict JSON schema).

```python
from typing import TypedDict

class UserData(TypedDict):
    name: str
    age: int

# Error if keys missing or types wrong
user: UserData = {"name": "Alice", "age": 30}
```

## `Protocol`
Structural subtyping (Duck Typing). See [Polymorphism](../../Intermediate/OOP/04_polymorphism.md) for details.
