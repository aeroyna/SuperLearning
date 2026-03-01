# Memory Management

Understanding Python's memory management helps write efficient code and debug memory issues.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Reference Counting**](01_reference_counting.md) | How Python tracks objects |
| [**2. Garbage Collection**](02_garbage_collection.md) | Cycle detection |
| [**3. Memory Optimization**](03_memory_optimization.md) | Reducing memory usage |

---

## Quick Reference

### Reference Counting
```python
import sys

a = [1, 2, 3]
print(sys.getrefcount(a))  # 2 (a + getrefcount's ref)

b = a  # Increase refcount
print(sys.getrefcount(a))  # 3

del b  # Decrease refcount
print(sys.getrefcount(a))  # 2
```

### Garbage Collection
```python
import gc

# Manual collection
gc.collect()

# Get stats
gc.get_count()  # (gen0, gen1, gen2)

# Disable/enable
gc.disable()
gc.enable()
```

### Weak References
```python
import weakref

class MyClass:
    pass

obj = MyClass()
ref = weakref.ref(obj)

print(ref())  # <__main__.MyClass object>
del obj
print(ref())  # None (object was collected)
```

---

## Memory Tools

### sys.getsizeof()
```python
import sys

sys.getsizeof([])      # 56 bytes
sys.getsizeof([1,2,3]) # 88 bytes
sys.getsizeof({})      # 64 bytes
```

### __slots__
```python
class PointNoSlots:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class PointWithSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

# PointWithSlots uses less memory
```

### Memory Profilers
```python
# memory_profiler
from memory_profiler import profile

@profile
def my_function():
    data = [i for i in range(100000)]
    return sum(data)

# tracemalloc
import tracemalloc

tracemalloc.start()
# ... your code ...
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
```

---

## Common Memory Issues

1. **Circular References** — Use weakref or break cycles
2. **Large Data Structures** — Use generators, itertools
3. **Global Variables** — Limit scope
4. **Caching** — Use LRU cache with maxsize

---

## Next Steps
Start with [Reference Counting](01_reference_counting.md).
