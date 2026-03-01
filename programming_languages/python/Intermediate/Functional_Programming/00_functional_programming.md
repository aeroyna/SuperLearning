# Functional Programming

Python supports functional programming concepts alongside OOP, enabling powerful data transformations and compositions.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. First-Class Functions**](01_first_class_functions.md) | Functions as objects, higher-order functions |
| [**2. Map, Filter, Reduce**](02_map_filter_reduce.md) | Functional transformations on iterables |
| [**3. Comprehensions**](03_comprehensions.md) | Pythonic functional expressions |
| [**4. Partial Functions**](04_partial_functions.md) | functools.partial and currying |
| [**5. Pure Functions**](05_pure_functions.md) | Side effects and immutability |

---

## Key Concepts

### First-Class Functions
```python
def greet(name):
    return f"Hello, {name}"

# Functions as objects
say_hi = greet
say_hi("World")  # "Hello, World"

# Functions as arguments
def apply(func, value):
    return func(value)

apply(greet, "Python")  # "Hello, Python"

# Functions as return values
def make_multiplier(n):
    def multiply(x):
        return x * n
    return multiply

double = make_multiplier(2)
double(5)  # 10
```

### Pure Functions
```python
# Pure: same input → same output, no side effects
def add(a, b):
    return a + b

# Impure: modifies external state
total = 0
def add_to_total(x):
    global total
    total += x
```

### Immutability
```python
# Instead of modifying
items = [1, 2, 3]
items.append(4)

# Create new collection
new_items = items + [4]
new_items = [*items, 4]
```

---

## Functional Tools

### map()
```python
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
# [1, 4, 9, 16, 25]

# Pythonic alternative
squared = [x ** 2 for x in numbers]
```

### filter()
```python
numbers = [1, 2, 3, 4, 5, 6]
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4, 6]

# Pythonic alternative
evens = [x for x in numbers if x % 2 == 0]
```

### reduce()
```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]
total = reduce(lambda acc, x: acc + x, numbers)
# 15

# Often clearer with built-in
total = sum(numbers)
```

---

## functools Module

```python
from functools import partial, lru_cache, reduce

# Partial application
def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
square(5)  # 25

# Memoization
@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci(100)  # Instant with caching
```

---

## itertools Module

```python
from itertools import chain, cycle, repeat, takewhile, dropwhile

# Chain iterables
list(chain([1, 2], [3, 4]))  # [1, 2, 3, 4]

# Infinite iterators
from itertools import count, islice
list(islice(count(10), 5))  # [10, 11, 12, 13, 14]

# Grouping
from itertools import groupby
data = [('a', 1), ('a', 2), ('b', 3), ('b', 4)]
for key, group in groupby(data, key=lambda x: x[0]):
    print(key, list(group))
```

---

## Next Steps
Start with [First-Class Functions](01_first_class_functions.md).
