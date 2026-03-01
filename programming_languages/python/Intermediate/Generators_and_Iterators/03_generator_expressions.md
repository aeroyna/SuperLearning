# Generator Expressions

Generator expressions are to generators what list comprehensions are to lists: a concise syntax for creating them efficiently.

## Syntax

Same as list comprehension but with **parentheses** `()`.

```python
# List Comprehension (Eager)
squares_list = [x**2 for x in range(10)]
# Creates full list in memory immediately

# Generator Expression (Lazy)
squares_gen = (x**2 for x in range(10))
# Creates generator object. Computes specific square only when requested.
```

## Comparisons

### Memory Usage
```python
import sys

# 1 Million numbers
my_list = [x for x in range(1000000)]
my_gen = (x for x in range(1000000))

print(sys.getsizeof(my_list)) # ~8.5 MB
print(sys.getsizeof(my_gen))  # ~120 Bytes
```

### Speed
*   **List**: Faster if you need to iterate multiple times (data is pre-calculated).
*   **Generator**: Faster startup time (doesn't compute anything initially).

## When to Use
*   Use **Generator** if:
    *   The sequence is infinite.
    *   The sequence is massive.
    *   You only iterate once (e.g., passing to `sum()`, `max()`).
*   Use **List** if:
    *   You need to iterate multiple times.
    *   You need random access (indexing).
