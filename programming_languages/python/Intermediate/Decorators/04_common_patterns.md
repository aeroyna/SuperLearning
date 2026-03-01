# Common Decorator Patterns

## 1. Timing Execution

```python
import time
import functools

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.perf_counter()
        value = func(*args, **kwargs)
        end_time = time.perf_counter()
        run_time = end_time - start_time
        print(f"Finished {func.__name__} in {run_time:.4f} secs")
        return value
    return wrapper
```

## 2. Debugging / Logging

```python
def debug(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        args_repr = [repr(a) for a in args]
        kwargs_repr = [f"{k}={v!r}" for k, v in kwargs.items()]
        signature = ", ".join(args_repr + kwargs_repr)
        print(f"Calling {func.__name__}({signature})")
        value = func(*args, **kwargs)
        print(f"{func.__name__!r} returned {value!r}")
        return value
    return wrapper
```

## 3. Registering Plugins
Decorators don't have to wrap logic; they can just register functions.

```python
PLUGINS = {}

def register(func):
    PLUGINS[func.__name__] = func
    return func  # Return original function unmodified

@register
def say_hello():
    return "Hello"

print(PLUGINS) # {'say_hello': <function say_hello ...>}
```

## 4. State Management (Classes as Decorators)
Sometimes it's cleaner to use a class as a decorator to maintain state.

```python
class CountCalls:
    def __init__(self, func):
        self.func = func
        self.num_calls = 0
        functools.update_wrapper(self, func)

    def __call__(self, *args, **kwargs):
        self.num_calls += 1
        print(f"Call {self.num_calls} of {self.func.__name__!r}")
        return self.func(*args, **kwargs)

@CountCalls
def say_hi():
    print("Hi!")
```
