# Performance Optimization

Learn to identify and fix performance bottlenecks in Python code.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Profiling**](01_profiling.md) | Measuring performance |
| [**2. Optimization Techniques**](02_optimization.md) | Common speedups |
| [**3. Cython and C Extensions**](03_cython.md) | Native performance |

---

## Quick Reference

### Timing
```python
import time
import timeit

# Basic timing
start = time.perf_counter()
# ... code ...
elapsed = time.perf_counter() - start

# timeit
timeit.timeit('sum(range(1000))', number=10000)

# In IPython/Jupyter
%timeit sum(range(1000))
```

### Profiling
```python
import cProfile
import pstats

# Profile code
cProfile.run('my_function()', 'output.prof')

# Analyze
stats = pstats.Stats('output.prof')
stats.sort_stats('cumtime')
stats.print_stats(10)

# Command line
# python -m cProfile -o output.prof script.py
```

---

## Common Optimizations

### Use Built-ins
```python
# Slow
total = 0
for x in items:
    total += x

# Fast
total = sum(items)
```

### List Comprehensions
```python
# Slow
result = []
for x in items:
    result.append(x * 2)

# Faster
result = [x * 2 for x in items]
```

### Local Variables
```python
# Slow
def slow():
    for i in range(1000):
        math.sqrt(i)

# Faster
def fast():
    sqrt = math.sqrt
    for i in range(1000):
        sqrt(i)
```

### Generators for Large Data
```python
# Memory heavy
squares = [x**2 for x in range(1000000)]

# Memory efficient
squares = (x**2 for x in range(1000000))
```

### Right Data Structure
```python
# O(n) lookup
items = [1, 2, 3, 4, 5]
3 in items

# O(1) lookup
items = {1, 2, 3, 4, 5}
3 in items
```

---

## Tools

| Tool | Purpose |
|------|---------|
| `cProfile` | Function-level profiling |
| `line_profiler` | Line-by-line profiling |
| `memory_profiler` | Memory usage |
| `py-spy` | Sampling profiler |
| `timeit` | Microbenchmarking |

---

## Next Steps
Start with [Profiling](01_profiling.md).
