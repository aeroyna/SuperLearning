# Structural Pattern Matching

Introduced in Python 3.10 (PEP 634), Structural Pattern Matching is more than just a `switch` statement. It allows you to match the **structure** of data (types, shapes, content) and deconstruct it.

## Basic Syntax

```python
match subject:
    case <pattern_1>:
        <action_1>
    case <pattern_2>:
        <action_2>
    case _:
        <default_action>
```

## Patterns

### 1. Literal Patterns
Matching specific values.

```python
status = 404

match status:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case _:
        print("Something else")
```

### 2. Sequence Patterns
Matching lists or tuples.

```python
point = (0, 0)

match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y-axis at {y}")
    case (x, 0):
        print(f"X-axis at {x}")
    case (x, y):
        print(f"Point at {x}, {y}")
```

### 3. Mapping Patterns
Matching dictionaries (checks for subset of keys).

```python
user = {"name": "Alice", "age": 30, "admin": True}

match user:
    case {"name": name, "admin": True}:
        print(f"Admin user: {name}")
    case {"name": name}:
        print(f"User: {name}")
```

### 4. Class Patterns
Matching objects and attributes.

```python
@dataclass
class Point:
    x: int
    y: int

def where_is(point):
    match point:
        case Point(x=0, y=0):
            print("Origin")
        case Point(x=0, y=y):
            print(f"Y-axis at {y}")
        case Point(x=x, y=0):
            print(f"X-axis at {x}")
        case Point():
            print("Somewhere else")
```

## Guards
You can add `if` conditions to patterns.

```python
match point:
    case Point(x, y) if x == y:
        print(f"Y=X at {x}")
```

## Best Practices
*   Use it for data parsing/routing.
*   Don't over-use it for simple `if/else` logic.
*   Remember `_` is the wildcard (default case).
