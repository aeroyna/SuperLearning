# Context Managers

Context managers handle setup and cleanup operations automatically, ensuring resources are properly managed.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Using Context Managers**](01_using_context_managers.md) | The with statement |
| [**2. Creating Context Managers**](02_creating_context_managers.md) | Class and contextlib approaches |

---

## Quick Reference

### Basic Usage
```python
with open('file.txt') as f:
    content = f.read()
# File automatically closed

# Multiple context managers
with open('input.txt') as src, open('output.txt', 'w') as dst:
    dst.write(src.read())
```

### Class-Based Context Manager
```python
class Timer:
    def __enter__(self):
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.time() - self.start
        print(f"Elapsed: {self.elapsed:.2f}s")
        return False  # Don't suppress exceptions

with Timer() as t:
    time.sleep(1)
# Elapsed: 1.00s
```

### @contextmanager Decorator
```python
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.time()
    yield
    elapsed = time.time() - start
    print(f"Elapsed: {elapsed:.2f}s")

with timer():
    time.sleep(1)
```

---

## Common Patterns

### Resource Management
```python
@contextmanager
def managed_resource():
    resource = acquire_resource()
    try:
        yield resource
    finally:
        release_resource(resource)
```

### Temporary State
```python
@contextmanager
def temporary_directory():
    import tempfile
    import shutil
    dir_path = tempfile.mkdtemp()
    try:
        yield dir_path
    finally:
        shutil.rmtree(dir_path)

with temporary_directory() as tmpdir:
    # Work with temporary directory
    pass
# Directory cleaned up
```

### Suppress Exceptions
```python
from contextlib import suppress

with suppress(FileNotFoundError):
    os.remove('nonexistent.txt')
# No error if file doesn't exist
```

---

## Next Steps
Start with [Using Context Managers](01_using_context_managers.md).
