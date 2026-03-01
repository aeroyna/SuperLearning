# Pure Functions

Pure functions are a core concept in functional programming. They have two key properties:
1.  **Deterministic**: Given the same input, they *always* return the same output.
2.  **No Side Effects**: They do not modify external state (variables, files, databases) or arguments.

## Examples

### Impure (Avoid)
```python
total = 0

def add_to_total(x):
    global total
    total += x  # Side effect: modifies external state
    return total

def append_to_list(lst, item):
    lst.append(item)  # Side effect: modifies argument
    return lst
```

### Pure (Preferred)
```python
def add(x, y):
    return x + y  # Depends only on inputs

def add_to_list(lst, item):
    return lst + [item]  # Returns NEW list, doesn't touch original
```

## Benefits of Purity

1.  **Testability**: Trivial to unit test. No setup/teardown of mocks or state needed.
2.  **Concurrency**: Truly pure functions are thread-safe by default (no shared mutable state).
3.  **Caching**: Results can be memoized easily.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def expensive_pure_function(x):
    # heavy computation
    return x * x
```

## Immutability in Python
Since Python objects are mutable by default (lists, dicts), writing pure functions requires discipline.
*   Use `tuple` instead of `list`.
*   Use `frozenset` instead of `set`.
*   Return copies of data structures.
