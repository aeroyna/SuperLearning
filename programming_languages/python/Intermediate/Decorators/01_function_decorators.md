# Function Decorators

Decorators are a powerful way to modify or enhance the behavior of functions or classes without changing their source code. They are essentially wrappers around a function.

## Basic Syntax

The `@decorator` syntax is syntactic sugar.

```python
@my_decorator
def my_function():
    pass

# Equivalent to:
my_function = my_decorator(my_function)
```

## Creating a Decorator

A decorator is a function that takes a function as an argument and returns a new function (the wrapper).

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

## Decorating Functions with Arguments

To support functions with arguments, the wrapper uses `*args` and `**kwargs`.

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}, {kwargs}")
        return func(*args, **kwargs)
    return wrapper

@logger
def add(x, y):
    return x + y

add(5, 3)
```

## Preserving Metadata (`functools.wraps`)
When you wrap a function, the new function (wrapper) loses the original name and docstring. Use `functools.wraps` to fix this.

```python
import functools

def my_decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

## Decorators taking Arguments
To pass arguments to the decorator itself (e.g., `@repeat(3)`), you need a three-level nested function.

```python
def repeat(num_times):
    def decorator_repeat(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(num_times):
                value = func(*args, **kwargs)
            return value
        return wrapper
    return decorator_repeat

@repeat(num_times=4)
def greet(name):
    print(f"Hello {name}")
```
