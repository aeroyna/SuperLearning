# Built-in Decorators

Python includes several powerful decorators in the standard library.

## `@property`
Transforms a method into a read-only attribute (getter). See [Encapsulation](../../Intermediate/OOP/03_encapsulation.md).

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def area(self):
        return 3.14 * self._radius ** 2
```

## `@classmethod` and `@staticmethod`
Define methods bound to the class or neither. See [Class vs Instance](../../Intermediate/OOP/06_class_vs_instance.md).

## `@functools.lru_cache`
Least Recently Used (LRU) cache. Automatically memoizes function results.

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)
```

## `@functools.total_ordering`
Given a class defining one or more rich comparison ordering methods (like `__lt__`), this class decorator supplies the rest.

```python
from functools import total_ordering

@total_ordering
class Student:
    def __init__(self, grade):
        self.grade = grade

    def __eq__(self, other):
        return self.grade == other.grade

    def __lt__(self, other):
        return self.grade < other.grade

# Now __le__, __gt__, __ge__ work automatically
```

## `@dataclasses.dataclass`
Automatically generates `__init__`, `__repr__`, `__eq__`, etc.

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
```
