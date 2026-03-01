# Decorators Preview

Decorators are a powerful pattern for modifying or enhancing functions. This is a brief introduction; see the full [Decorators](../../Intermediate/Decorators/00_decorators.md) section for comprehensive coverage.

---

## 1. What is a Decorator?

A decorator is a function that takes a function and returns a modified version:

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before function call")
        result = func(*args, **kwargs)
        print("After function call")
        return result
    return wrapper

@my_decorator
def say_hello(name):
    print(f"Hello, {name}!")

say_hello("Alice")
# Before function call
# Hello, Alice!
# After function call
```

### Without `@` Syntax
```python
# This:
@my_decorator
def say_hello(name):
    print(f"Hello, {name}!")

# Is equivalent to:
def say_hello(name):
    print(f"Hello, {name}!")
say_hello = my_decorator(say_hello)
```

---

## 2. Common Use Cases

### Timing Functions
```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

slow_function()
# slow_function took 1.0012 seconds
```

### Logging
```python
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_call
def add(a, b):
    return a + b

add(3, 5)
# Calling add with (3, 5), {}
# add returned 8
```

### Authentication
```python
def require_auth(func):
    def wrapper(*args, **kwargs):
        if not is_authenticated():
            raise PermissionError("Not authenticated")
        return func(*args, **kwargs)
    return wrapper

@require_auth
def get_secret_data():
    return "Top secret!"
```

---

## 3. Preserving Function Metadata

Use `functools.wraps` to preserve the original function's metadata:

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # Preserve func's name and docstring
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)  # 'greet' (not 'wrapper')
print(greet.__doc__)   # 'Greet someone by name.'
```

---

## 4. Decorators with Arguments

```python
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def say_hello(name):
    print(f"Hello, {name}!")

say_hello("Alice")
# Hello, Alice!
# Hello, Alice!
# Hello, Alice!
```

---

## 5. Stacking Decorators

```python
@decorator1
@decorator2
@decorator3
def func():
    pass

# Equivalent to:
func = decorator1(decorator2(decorator3(func)))
```

---

## 6. Built-in Decorators

### `@property`
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

c = Circle(5)
print(c.radius)  # 5
c.radius = 10    # Uses setter
```

### `@staticmethod` and `@classmethod`
```python
class MyClass:
    class_var = 0

    @staticmethod
    def static_method():
        return "I don't access instance or class"

    @classmethod
    def class_method(cls):
        return f"Class var is {cls.class_var}"
```

### `@functools.lru_cache`
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci(100)  # Fast due to caching
```

---

## 7. Quick Reference

```python
# Simple decorator
def decorator(func):
    def wrapper(*args, **kwargs):
        # Before
        result = func(*args, **kwargs)
        # After
        return result
    return wrapper

# Decorator with arguments
def decorator_with_args(arg):
    def decorator(func):
        def wrapper(*args, **kwargs):
            # Use arg
            return func(*args, **kwargs)
        return wrapper
    return decorator

# Class-based decorator
class Decorator:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)
```

---

## Next Steps

For comprehensive decorator coverage including:
- Class decorators
- Decorator factories
- Common patterns
- Best practices

See [Decorators](../../Intermediate/Decorators/00_decorators.md).
