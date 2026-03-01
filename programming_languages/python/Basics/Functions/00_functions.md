# Functions

Functions are the fundamental building blocks for organizing and reusing code in Python.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Function Basics**](01_function_basics.md) | Definition, calling, return values |
| [**2. Arguments and Parameters**](02_arguments.md) | Positional, keyword, default, *args, **kwargs |
| [**3. Scope and Closures**](03_scope_and_closures.md) | LEGB rule, global, nonlocal, closures |
| [**4. Lambda Functions**](04_lambda_functions.md) | Anonymous functions and functional programming |
| [**5. Decorators Preview**](05_decorators_preview.md) | Introduction to decorators |

---

## Quick Reference

### Basic Function
```python
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

result = greet("World")  # "Hello, World!"
```

### Default Arguments
```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")            # "Hello, Alice!"
greet("Alice", "Hi")      # "Hi, Alice!"
```

### Variable Arguments
```python
def sum_all(*args):
    return sum(args)

sum_all(1, 2, 3, 4)  # 10

def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=30)
```

### Lambda Functions
```python
square = lambda x: x ** 2
add = lambda x, y: x + y

# With higher-order functions
sorted(items, key=lambda x: x.name)
list(filter(lambda x: x > 0, numbers))
list(map(lambda x: x * 2, numbers))
```

---

## Function Types in Python

| Type | Example |
|------|---------|
| Regular function | `def func(): ...` |
| Lambda function | `lambda x: x * 2` |
| Method | `def method(self): ...` |
| Built-in function | `len()`, `print()`, `sum()` |
| Generator function | `def gen(): yield x` |
| Async function | `async def func(): ...` |

---

## Key Concepts

### Functions Are First-Class Objects
```python
def greet(name):
    return f"Hello, {name}!"

# Assign to variable
f = greet
f("World")  # "Hello, World!"

# Pass to function
def apply(func, value):
    return func(value)

apply(greet, "Python")  # "Hello, Python!"

# Store in data structure
functions = [greet, str.upper, len]
```

### Docstrings
```python
def calculate_area(radius):
    """
    Calculate the area of a circle.

    Args:
        radius: The radius of the circle.

    Returns:
        The area of the circle.

    Raises:
        ValueError: If radius is negative.
    """
    if radius < 0:
        raise ValueError("Radius cannot be negative")
    return 3.14159 * radius ** 2

# Access docstring
print(calculate_area.__doc__)
help(calculate_area)
```

---

## Next Steps
Start with [Function Basics](01_function_basics.md).
