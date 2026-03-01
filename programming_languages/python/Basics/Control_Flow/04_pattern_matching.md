# Pattern Matching (Python 3.10+)

Structural pattern matching, introduced in Python 3.10, provides powerful ways to match values against patterns.

---

## 1. Basic `match` Statement

```python
def http_status(status):
    match status:
        case 200:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Internal Server Error"
        case _:
            return "Unknown"

http_status(200)  # "OK"
http_status(999)  # "Unknown"
```

The `_` pattern is a wildcard that matches anything.

---

## 2. Literal Patterns

```python
def describe(value):
    match value:
        case 0:
            return "zero"
        case 1:
            return "one"
        case True:  # Note: True == 1, but matched separately
            return "true boolean"
        case "hello":
            return "greeting"
        case None:
            return "nothing"
        case _:
            return "something else"
```

---

## 3. Capture Patterns

Capture matched values into variables:

```python
def describe(point):
    match point:
        case (0, 0):
            return "Origin"
        case (0, y):
            return f"On Y-axis at {y}"
        case (x, 0):
            return f"On X-axis at {x}"
        case (x, y):
            return f"Point at ({x}, {y})"
        case _:
            return "Not a point"

describe((0, 0))    # "Origin"
describe((0, 5))    # "On Y-axis at 5"
describe((3, 4))    # "Point at (3, 4)"
```

---

## 4. Sequence Patterns

### Fixed Length
```python
def process(data):
    match data:
        case []:
            return "Empty"
        case [single]:
            return f"Single element: {single}"
        case [first, second]:
            return f"Two elements: {first}, {second}"
        case [first, second, third]:
            return f"Three elements"
        case _:
            return "Many elements"
```

### Variable Length with `*`
```python
def process(data):
    match data:
        case [first, *rest]:
            return f"First: {first}, Rest: {rest}"
        case [*init, last]:
            return f"Last: {last}"
        case [first, *middle, last]:
            return f"First: {first}, Middle: {middle}, Last: {last}"

process([1, 2, 3, 4, 5])
# "First: 1, Rest: [2, 3, 4, 5]"
```

---

## 5. Mapping Patterns (Dictionaries)

```python
def process_action(action):
    match action:
        case {"type": "click", "x": x, "y": y}:
            return f"Click at ({x}, {y})"
        case {"type": "keypress", "key": key}:
            return f"Key pressed: {key}"
        case {"type": "scroll", "direction": direction}:
            return f"Scroll {direction}"
        case _:
            return "Unknown action"

process_action({"type": "click", "x": 100, "y": 200})
# "Click at (100, 200)"

# Extra keys are ignored
process_action({"type": "click", "x": 100, "y": 200, "button": "left"})
# Still matches!
```

### Capture Remaining Keys
```python
def process(data):
    match data:
        case {"name": name, **rest}:
            return f"Name: {name}, Other: {rest}"

process({"name": "Alice", "age": 30, "city": "NYC"})
# "Name: Alice, Other: {'age': 30, 'city': 'NYC'}"
```

---

## 6. Class Patterns

Match against class instances:

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

@dataclass
class Circle:
    center: Point
    radius: float

def describe(shape):
    match shape:
        case Point(x=0, y=0):
            return "Origin"
        case Point(x=0, y=y):
            return f"On Y-axis at {y}"
        case Point(x=x, y=0):
            return f"On X-axis at {x}"
        case Point(x=x, y=y):
            return f"Point at ({x}, {y})"
        case Circle(center=Point(x=0, y=0), radius=r):
            return f"Circle at origin with radius {r}"
        case Circle(center=c, radius=r):
            return f"Circle at {c} with radius {r}"

describe(Point(0, 5))
# "On Y-axis at 5"

describe(Circle(Point(0, 0), 10))
# "Circle at origin with radius 10"
```

### Positional Patterns with `__match_args__`
```python
class Point:
    __match_args__ = ("x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

def describe(point):
    match point:
        case Point(0, 0):
            return "Origin"
        case Point(0, y):
            return f"On Y-axis"
        case Point(x, y):
            return f"Point at ({x}, {y})"
```

---

## 7. OR Patterns (`|`)

Match multiple patterns:

```python
def describe(status):
    match status:
        case 200 | 201 | 204:
            return "Success"
        case 400 | 401 | 403 | 404:
            return "Client Error"
        case 500 | 502 | 503:
            return "Server Error"
        case _:
            return "Unknown"

# With capture (must use same name in all alternatives)
def parse_command(cmd):
    match cmd.split():
        case ["quit" | "exit" | "q"]:
            return "Exiting..."
        case ["load" | "open", filename]:
            return f"Loading {filename}"
```

---

## 8. Guards (Conditional Patterns)

Add conditions with `if`:

```python
def describe(point):
    match point:
        case (x, y) if x == y:
            return f"On diagonal at {x}"
        case (x, y) if x > 0 and y > 0:
            return f"In first quadrant"
        case (x, y) if x < 0 and y > 0:
            return f"In second quadrant"
        case (x, y):
            return f"Point at ({x}, {y})"

describe((5, 5))   # "On diagonal at 5"
describe((3, 4))   # "In first quadrant"
describe((-1, 2))  # "In second quadrant"
```

---

## 9. AS Patterns

Capture entire matched value:

```python
def process(data):
    match data:
        case {"name": str(name), "age": int(age)} as person:
            return f"Valid person: {person}"
        case _:
            return "Invalid"

process({"name": "Alice", "age": 30})
# "Valid person: {'name': 'Alice', 'age': 30}"
```

---

## 10. Type Patterns

Match by type:

```python
def process(value):
    match value:
        case str(s):
            return f"String of length {len(s)}"
        case int(n):
            return f"Integer: {n}"
        case list(items):
            return f"List with {len(items)} items"
        case dict(d):
            return f"Dict with keys: {list(d.keys())}"
        case _:
            return "Unknown type"

process("hello")      # "String of length 5"
process(42)           # "Integer: 42"
process([1, 2, 3])    # "List with 3 items"
```

---

## 11. Real-World Examples

### Command Parser
```python
def parse_command(command):
    match command.split():
        case ["quit"]:
            return ("quit",)
        case ["help"]:
            return ("help",)
        case ["load", filename]:
            return ("load", filename)
        case ["save", filename]:
            return ("save", filename)
        case ["move", x, y]:
            return ("move", int(x), int(y))
        case ["set", key, value]:
            return ("set", key, value)
        case [cmd, *args]:
            return ("unknown", cmd, args)
```

### JSON API Response Handler
```python
def handle_response(response):
    match response:
        case {"status": "success", "data": data}:
            return process_data(data)
        case {"status": "error", "message": msg, "code": code}:
            raise APIError(code, msg)
        case {"status": "error", "message": msg}:
            raise APIError(500, msg)
        case _:
            raise ValueError("Invalid response format")
```

### AST Processing
```python
def evaluate(expr):
    match expr:
        case int(n) | float(n):
            return n
        case ("+", left, right):
            return evaluate(left) + evaluate(right)
        case ("-", left, right):
            return evaluate(left) - evaluate(right)
        case ("*", left, right):
            return evaluate(left) * evaluate(right)
        case ("/", left, right):
            return evaluate(left) / evaluate(right)
        case _:
            raise ValueError(f"Unknown expression: {expr}")

evaluate(("+", 1, ("*", 2, 3)))  # 7
```

---

## 12. Best Practices

### Order Matters
```python
# More specific patterns first
match point:
    case (0, 0):       # Most specific
        return "origin"
    case (0, y):       # Less specific
        return "y-axis"
    case (x, y):       # Least specific (catch-all)
        return "point"
```

### Always Include a Catch-All
```python
match value:
    case specific_pattern:
        ...
    case _:
        raise ValueError(f"Unexpected value: {value}")
```

### Use Type Guards for Safety
```python
case int(n) if n > 0:    # Positive integer
case str(s) if s:        # Non-empty string
case list(items) if items:  # Non-empty list
```

---

## 13. Practice Problems

1. Write a calculator using pattern matching that handles nested expressions.

2. Parse a simple SQL-like query: `SELECT name FROM users WHERE age > 30`.

3. Implement a state machine for a vending machine using pattern matching.

---

## Summary

Pattern matching provides:
- **Readable conditional logic** for complex data structures
- **Destructuring** with capture variables
- **Type checking** built into patterns
- **Guards** for additional conditions

It's especially useful for:
- Command parsing
- API response handling
- AST/tree processing
- State machines
- Data validation
