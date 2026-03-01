# Creating Context Managers

You can create your own context managers to handle setup/teardown logic cleanly.

## Class-Based (`__enter__`, `__exit__`)

```python
class Timer:
    def __enter__(self):
        import time
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.end = time.time()
        print(f"Elapsed: {self.end - self.start}s")
        # Return False to propagate exceptions, True to suppress
        return False

with Timer():
    for _ in range(1000000): pass
```

### The `__exit__` Arguments
*   `exc_type`: Exception class
*   `exc_val`: Exception instance
*   `exc_tb`: Traceback object
If no exception occurred, all are `None`.

## Generator-Based (`@contextmanager`)

Using `contextlib`, you can turn a generator into a context manager. Code before `yield` is setup; code after is teardown.

```python
from contextlib import contextmanager
import os

@contextmanager
def change_dir(destination):
    cwd = os.getcwd()
    os.chdir(destination)
    try:
        yield
    finally:
        os.chdir(cwd)

with change_dir('/tmp'):
    print(os.getcwd()) # /tmp
print(os.getcwd()) # Original path
```
This is often more concise than defining a class.
