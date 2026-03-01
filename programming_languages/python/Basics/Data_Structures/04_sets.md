# Sets

Sets are unordered collections of unique, hashable elements with O(1) membership testing.

---

## 1. Creating Sets

```python
# Empty set (not {} — that's a dict!)
empty = set()

# With elements
fruits = {"apple", "banana", "cherry"}
numbers = {1, 2, 3, 4, 5}

# Duplicates are ignored
{1, 2, 2, 3, 3, 3}  # {1, 2, 3}

# From other iterables
set([1, 2, 3])        # {1, 2, 3}
set("hello")          # {'h', 'e', 'l', 'o'}
set(range(5))         # {0, 1, 2, 3, 4}

# Set comprehension
squares = {x**2 for x in range(5)}  # {0, 1, 4, 9, 16}
```

---

## 2. Immutable Sets: frozenset

```python
# Frozenset is immutable
fs = frozenset([1, 2, 3])
# fs.add(4)  # AttributeError

# Can be used as dict key or set element
d = {frozenset({1, 2}): "value"}
s = {frozenset({1, 2}), frozenset({3, 4})}
```

---

## 3. Basic Operations

### Adding Elements
```python
s = {1, 2, 3}

s.add(4)          # {1, 2, 3, 4}
s.add(3)          # {1, 2, 3, 4} — no duplicate

s.update([5, 6])  # {1, 2, 3, 4, 5, 6}
s.update({7}, [8, 9])  # Add from multiple iterables
```

### Removing Elements
```python
s = {1, 2, 3, 4, 5}

s.remove(3)       # Remove 3 (raises KeyError if not found)
s.discard(10)     # Remove if exists (no error if not found)
s.pop()           # Remove and return arbitrary element
s.clear()         # Remove all elements
```

### Membership Testing
```python
s = {1, 2, 3}

2 in s        # True — O(1)
5 not in s    # True — O(1)
```

---

## 4. Set Operations

### Union (elements in either set)
```python
a = {1, 2, 3}
b = {3, 4, 5}

a | b           # {1, 2, 3, 4, 5}
a.union(b)      # {1, 2, 3, 4, 5}
a |= b          # a = {1, 2, 3, 4, 5} (in-place)
```

### Intersection (elements in both sets)
```python
a = {1, 2, 3}
b = {2, 3, 4}

a & b                  # {2, 3}
a.intersection(b)      # {2, 3}
a &= b                 # a = {2, 3} (in-place)
```

### Difference (elements in first but not second)
```python
a = {1, 2, 3}
b = {2, 3, 4}

a - b               # {1}
a.difference(b)     # {1}
a -= b              # a = {1} (in-place)
```

### Symmetric Difference (elements in either but not both)
```python
a = {1, 2, 3}
b = {2, 3, 4}

a ^ b                        # {1, 4}
a.symmetric_difference(b)    # {1, 4}
a ^= b                       # a = {1, 4} (in-place)
```

---

## 5. Set Comparisons

```python
a = {1, 2, 3}
b = {1, 2, 3, 4, 5}
c = {1, 2, 3}

# Subset
a <= b              # True (a is subset of b)
a.issubset(b)       # True
a < b               # True (proper subset)

# Superset
b >= a              # True (b is superset of a)
b.issuperset(a)     # True
b > a               # True (proper superset)

# Equality
a == c              # True (same elements)

# Disjoint (no common elements)
{1, 2}.isdisjoint({3, 4})  # True
{1, 2}.isdisjoint({2, 3})  # False
```

---

## 6. Common Patterns

### Remove Duplicates
```python
lst = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(lst))  # [1, 2, 3, 4] (order may change)

# Preserve order (Python 3.7+)
unique = list(dict.fromkeys(lst))  # [1, 2, 2, 3, 3, 3, 4] -> [1, 2, 3, 4]
```

### Fast Membership Testing
```python
# Slow: O(n) for each lookup
valid_items = ["a", "b", "c", "d", "e"]
for item in data:
    if item in valid_items:  # O(n)
        process(item)

# Fast: O(1) for each lookup
valid_items = {"a", "b", "c", "d", "e"}
for item in data:
    if item in valid_items:  # O(1)
        process(item)
```

### Find Common Elements
```python
users_a = {"alice", "bob", "charlie"}
users_b = {"bob", "charlie", "david"}

common = users_a & users_b  # {"bob", "charlie"}
```

### Find Unique Elements
```python
list1 = [1, 2, 3, 4]
list2 = [3, 4, 5, 6]

# Only in list1
set(list1) - set(list2)  # {1, 2}

# Only in list2
set(list2) - set(list1)  # {5, 6}

# In either but not both
set(list1) ^ set(list2)  # {1, 2, 5, 6}
```

### Validate Allowed Values
```python
VALID_STATUSES = {"pending", "active", "completed", "cancelled"}

def set_status(status):
    if status not in VALID_STATUSES:
        raise ValueError(f"Invalid status: {status}")
    # ...
```

---

## 7. Set Methods Summary

| Method | Description |
|--------|-------------|
| `add(elem)` | Add element |
| `remove(elem)` | Remove element (raises KeyError) |
| `discard(elem)` | Remove element (no error) |
| `pop()` | Remove and return arbitrary element |
| `clear()` | Remove all elements |
| `union(other)` | Return union |
| `intersection(other)` | Return intersection |
| `difference(other)` | Return difference |
| `symmetric_difference(other)` | Return symmetric difference |
| `update(other)` | Add elements from other |
| `issubset(other)` | Test if subset |
| `issuperset(other)` | Test if superset |
| `isdisjoint(other)` | Test if no common elements |
| `copy()` | Return shallow copy |

---

## 8. Set Comprehension

```python
# Basic
evens = {x for x in range(10) if x % 2 == 0}  # {0, 2, 4, 6, 8}

# Transform and filter
lengths = {len(word) for word in words if len(word) > 3}

# From nested structure
all_tags = {tag for post in posts for tag in post.tags}
```

---

## 9. Performance

| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Add | O(1) | O(n) |
| Remove | O(1) | O(n) |
| `in` check | O(1) | O(n) |
| Union | O(len(s) + len(t)) | — |
| Intersection | O(min(len(s), len(t))) | — |

---

## 10. Sets vs Other Types

| Need | Use |
|------|-----|
| Fast lookup, no duplicates | `set` |
| Ordered, allow duplicates | `list` |
| Immutable, as dict key | `frozenset` |
| Key-value pairs | `dict` |

---

## 11. Practice Problems

1. Find all common elements among three or more lists.

2. Implement a simple tag system where each item can have multiple tags.

3. Find elements that appear in exactly k out of n lists.

4. Check if two strings are anagrams using sets.

---

## Next Steps
- [Comprehensions](05_comprehensions.md)
