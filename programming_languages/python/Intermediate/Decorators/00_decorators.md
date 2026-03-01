# Decorators

Decorators are a powerful feature for modifying or extending function and class behavior without changing their code.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Function Decorators**](01_function_decorators.md) | Basic decorators, with arguments |
| [**2. Class Decorators**](02_class_decorators.md) | Decorating and creating classes |
| [**3. Built-in Decorators**](03_builtin_decorators.md) | @property, @staticmethod, @classmethod |
| [**4. Common Patterns**](04_common_patterns.md) | Logging, timing, caching, validation |

---

## Quick Reference

### Basic Decorator
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Before
# Hello!
# After
```

### With functools.wraps
```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # Preserve function metadata
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### Decorator with Arguments
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
def greet(name):
    print(f"Hello, {name}!")

greet("World")  # Prints 3 times
```

---

## Common Decorators

### Timing
```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f} seconds")
        return result
    return wrapper
```

### Caching
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

### Retry
```python
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator
```

---

## Stacking Decorators

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

## Class-Based Decorators

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.func(*args, **kwargs)

@CountCalls
def say_hello():
    print("Hello!")

say_hello()
say_hello()
print(say_hello.count)  # 2
```

---

## Next Steps
Start with [Function Decorators](01_function_decorators.md).
