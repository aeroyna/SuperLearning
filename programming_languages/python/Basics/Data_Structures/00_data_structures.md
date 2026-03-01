# Built-in Data Structures

Python provides four primary built-in collection types, each with different characteristics and use cases.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Lists**](01_lists.md) | Ordered, mutable sequences |
| [**2. Tuples**](02_tuples.md) | Ordered, immutable sequences |
| [**3. Dictionaries**](03_dictionaries.md) | Key-value mappings |
| [**4. Sets**](04_sets.md) | Unordered collections of unique elements |
| [**5. Comprehensions**](05_comprehensions.md) | Concise syntax for creating collections |

---

## Quick Comparison

| Feature | List | Tuple | Dict | Set |
|---------|------|-------|------|-----|
| Syntax | `[1, 2, 3]` | `(1, 2, 3)` | `{'a': 1}` | `{1, 2, 3}` |
| Ordered | Yes | Yes | Yes (3.7+) | No |
| Mutable | Yes | No | Yes | Yes |
| Duplicates | Yes | Yes | Keys: No | No |
| Indexable | Yes | Yes | By key | No |
| Hashable | No | Yes* | No | No |

*Tuples are hashable only if all elements are hashable.

---

## When to Use Which

### List
- Ordered collection of items
- Need to modify elements
- Duplicate values allowed
- Example: task list, log entries

```python
tasks = ["buy milk", "write code", "buy milk"]
tasks.append("review PR")
tasks[0] = "buy eggs"
```

### Tuple
- Fixed collection of items
- Data shouldn't change
- Return multiple values from function
- Dictionary keys (if hashable)

```python
point = (3, 4)
rgb = (255, 128, 64)
x, y = point  # Unpacking

# As dict key
locations = {(0, 0): "origin", (1, 0): "right"}
```

### Dictionary
- Key-value associations
- Fast lookup by key
- JSON-like data structures
- Counting, grouping

```python
user = {"name": "Alice", "age": 30}
word_count = {"hello": 5, "world": 3}
```

### Set
- Unique elements only
- Fast membership testing
- Mathematical set operations
- Removing duplicates

```python
unique_words = {"apple", "banana", "apple"}  # {'apple', 'banana'}
if "apple" in unique_words:  # O(1) lookup
    print("Found!")
```

---

## Performance Characteristics

| Operation | List | Dict | Set |
|-----------|------|------|-----|
| Access by index | O(1) | — | — |
| Access by key | — | O(1) | — |
| Search | O(n) | O(1) | O(1) |
| Insert at end | O(1)* | O(1)* | O(1)* |
| Insert at start | O(n) | — | — |
| Delete | O(n) | O(1) | O(1) |

*Amortized

---

## Memory Considerations

```python
import sys

# Approximate memory usage (bytes)
sys.getsizeof([])         # 56
sys.getsizeof(())         # 40
sys.getsizeof({})         # 64
sys.getsizeof(set())      # 216

# Lists with elements
sys.getsizeof([1, 2, 3])  # 88

# Tuples are slightly more memory-efficient
sys.getsizeof((1, 2, 3))  # 64
```

---

## Common Patterns

### Converting Between Types
```python
# List to other types
lst = [1, 2, 2, 3]
tuple(lst)     # (1, 2, 2, 3)
set(lst)       # {1, 2, 3} — removes duplicates

# Dict from pairs
pairs = [('a', 1), ('b', 2)]
dict(pairs)    # {'a': 1, 'b': 2}

# List of dict keys/values
d = {'a': 1, 'b': 2}
list(d.keys())    # ['a', 'b']
list(d.values())  # [1, 2]
list(d.items())   # [('a', 1), ('b', 2)]
```

### Checking Emptiness
```python
# Use truthiness
if lst:
    print("List has items")

if not d:
    print("Dict is empty")
```

### Copying
```python
# Shallow copy
lst2 = lst.copy()      # or lst[:]
d2 = d.copy()
s2 = s.copy()

# Deep copy (for nested structures)
import copy
deep = copy.deepcopy(nested_list)
```

---

## Next Steps
Start with [Lists](01_lists.md) for the most commonly used collection type.
