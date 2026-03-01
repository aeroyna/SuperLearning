# Generators and Iterators

Generators and iterators are fundamental to Python's iteration protocol, enabling memory-efficient processing of sequences.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Iterators**](01_iterators.md) | Iterator protocol, custom iterators |
| [**2. Generators**](02_generators.md) | yield, generator functions |
| [**3. Generator Expressions**](03_generator_expressions.md) | Lazy evaluation |
| [**4. Advanced Generators**](04_advanced_generators.md) | send(), throw(), close() |

---

## Quick Reference

### Iterator Protocol
```python
# Any object with __iter__ and __next__
class Counter:
    def __init__(self, limit):
        self.limit = limit
        self.count = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.count >= self.limit:
            raise StopIteration
        self.count += 1
        return self.count

for n in Counter(5):
    print(n)  # 1, 2, 3, 4, 5
```

### Generator Function
```python
def count_up_to(limit):
    count = 1
    while count <= limit:
        yield count
        count += 1

for n in count_up_to(5):
    print(n)  # 1, 2, 3, 4, 5
```

### Generator Expression
```python
# Lazy evaluation
squares = (x**2 for x in range(1000000))
# No memory allocated for all values

# Use when iterating once
sum(x**2 for x in range(1000000))
```

---

## Why Generators?

### Memory Efficiency
```python
# List: stores all values
def squares_list(n):
    return [x**2 for x in range(n)]

# Generator: yields one at a time
def squares_gen(n):
    for x in range(n):
        yield x**2

# Memory usage
import sys
sys.getsizeof(squares_list(1000))   # ~9000 bytes
sys.getsizeof(squares_gen(1000))    # ~112 bytes
```

### Infinite Sequences
```python
def infinite_counter():
    n = 0
    while True:
        yield n
        n += 1

# Use with islice
from itertools import islice
list(islice(infinite_counter(), 10))  # [0, 1, 2, ..., 9]
```

---

## Common Patterns

### Pipeline
```python
def read_lines(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()

def filter_comments(lines):
    for line in lines:
        if not line.startswith('#'):
            yield line

def parse_values(lines):
    for line in lines:
        yield int(line)

# Compose generators
lines = read_lines('data.txt')
non_comments = filter_comments(lines)
values = parse_values(non_comments)

for value in values:
    process(value)
```

### yield from
```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

list(flatten([1, [2, 3, [4, 5]], 6]))
# [1, 2, 3, 4, 5, 6]
```

---

## Next Steps
Start with [Iterators](01_iterators.md).
