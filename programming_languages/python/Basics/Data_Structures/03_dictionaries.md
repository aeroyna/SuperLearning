# Dictionaries

Dictionaries are mutable mappings from keys to values, providing O(1) average-case lookup.

---

## 1. Creating Dictionaries

```python
# Empty dict
empty = {}
empty = dict()

# With elements
person = {"name": "Alice", "age": 30}
scores = {"math": 95, "english": 87, "science": 92}

# From key-value pairs
dict([("a", 1), ("b", 2)])  # {'a': 1, 'b': 2}

# From keyword arguments
dict(name="Alice", age=30)  # {'name': 'Alice', 'age': 30}

# From keys with default value
dict.fromkeys(["a", "b", "c"], 0)  # {'a': 0, 'b': 0, 'c': 0}

# Dict comprehension
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

---

## 2. Accessing Values

```python
d = {"name": "Alice", "age": 30, "city": "NYC"}

# Direct access
d["name"]        # "Alice"
# d["email"]     # KeyError: 'email'

# get() — safe access with default
d.get("name")          # "Alice"
d.get("email")         # None
d.get("email", "N/A")  # "N/A"

# Check existence
"name" in d       # True
"email" not in d  # True
```

---

## 3. Modifying Dictionaries

### Adding/Updating
```python
d = {"a": 1, "b": 2}

d["c"] = 3          # Add new key
d["a"] = 10         # Update existing

d.update({"b": 20, "d": 4})  # Update multiple
d.update(e=5)                 # Using kwargs

# Merge operators (Python 3.9+)
d | {"f": 6}        # New dict with merge
d |= {"f": 6}       # Update in place
```

### Removing
```python
d = {"a": 1, "b": 2, "c": 3}

del d["a"]          # Remove key
value = d.pop("b")  # Remove and return value
value = d.pop("z", None)  # With default if missing

d.clear()           # Remove all items

# Remove and return last item (Python 3.7+)
d = {"a": 1, "b": 2}
key, value = d.popitem()  # ('b', 2)
```

### setdefault()
```python
d = {"a": 1}

# Return value if exists, else set and return default
d.setdefault("a", 100)  # 1 (exists, returns current)
d.setdefault("b", 100)  # 100 (doesn't exist, sets and returns)
print(d)  # {'a': 1, 'b': 100}

# Common pattern: grouping
groups = {}
for item in items:
    groups.setdefault(item.category, []).append(item)
```

---

## 4. Iterating

```python
d = {"a": 1, "b": 2, "c": 3}

# Keys (default)
for key in d:
    print(key)

for key in d.keys():
    print(key)

# Values
for value in d.values():
    print(value)

# Key-value pairs
for key, value in d.items():
    print(f"{key}: {value}")

# Sorted iteration
for key in sorted(d.keys()):
    print(f"{key}: {d[key]}")
```

---

## 5. Dict Methods

```python
d = {"a": 1, "b": 2, "c": 3}

# Views (reflect changes to dict)
d.keys()    # dict_keys(['a', 'b', 'c'])
d.values()  # dict_values([1, 2, 3])
d.items()   # dict_items([('a', 1), ('b', 2), ('c', 3)])

# Copy
d.copy()    # Shallow copy

# Length
len(d)      # 3
```

---

## 6. Dictionary Comprehension

```python
# Basic
squares = {x: x**2 for x in range(5)}

# With condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}

# Transform dict
prices = {"apple": 1.00, "banana": 0.50}
discounted = {k: v * 0.9 for k, v in prices.items()}

# Swap keys and values
inverted = {v: k for k, v in d.items()}

# From two lists
keys = ["a", "b", "c"]
values = [1, 2, 3]
d = {k: v for k, v in zip(keys, values)}
```

---

## 7. Common Patterns

### Counting
```python
from collections import Counter

# Manual counting
text = "hello world"
count = {}
for char in text:
    count[char] = count.get(char, 0) + 1

# Using Counter
count = Counter(text)
count.most_common(3)  # [('l', 3), ('o', 2), ('h', 1)]
```

### Grouping
```python
from collections import defaultdict

# Manual grouping
groups = {}
for item in items:
    key = item.category
    if key not in groups:
        groups[key] = []
    groups[key].append(item)

# Using setdefault
groups = {}
for item in items:
    groups.setdefault(item.category, []).append(item)

# Using defaultdict
groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)
```

### Default Values with defaultdict
```python
from collections import defaultdict

# Default to 0
counts = defaultdict(int)
counts["a"] += 1  # No KeyError

# Default to empty list
groups = defaultdict(list)
groups["key"].append("value")

# Custom default
def default_value():
    return {"count": 0, "total": 0}

data = defaultdict(default_value)
```

### Merging Dictionaries
```python
d1 = {"a": 1, "b": 2}
d2 = {"b": 3, "c": 4}

# Python 3.9+
merged = d1 | d2  # {'a': 1, 'b': 3, 'c': 4}

# Python 3.5+
merged = {**d1, **d2}

# Older Python
merged = d1.copy()
merged.update(d2)
```

### Nested Dictionaries
```python
users = {
    "alice": {"age": 30, "email": "alice@example.com"},
    "bob": {"age": 25, "email": "bob@example.com"},
}

users["alice"]["age"]  # 30

# Safe nested access
def get_nested(d, *keys, default=None):
    for key in keys:
        if isinstance(d, dict):
            d = d.get(key, default)
        else:
            return default
    return d

get_nested(users, "alice", "age")  # 30
get_nested(users, "alice", "phone", default="N/A")  # "N/A"
```

---

## 8. Ordered Dictionaries

Since Python 3.7, regular dicts maintain insertion order. For older code or explicit ordering:

```python
from collections import OrderedDict

od = OrderedDict()
od["first"] = 1
od["second"] = 2

# Move to end
od.move_to_end("first")

# Move to beginning
od.move_to_end("second", last=False)
```

---

## 9. Performance

| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Get item | O(1) | O(n) |
| Set item | O(1) | O(n) |
| Delete item | O(1) | O(n) |
| Iteration | O(n) | O(n) |
| `in` check | O(1) | O(n) |

Worst case is rare (hash collisions).

---

## 10. Dictionary Keys

Keys must be hashable (immutable):

```python
# Valid keys
d = {
    "string": 1,
    42: 2,
    (1, 2): 3,          # Tuple
    frozenset({1, 2}): 4,
}

# Invalid keys
# d[[1, 2]] = 5  # TypeError: unhashable type: 'list'
# d[{1, 2}] = 5  # TypeError: unhashable type: 'set'
```

---

## 11. Practice Problems

1. Write a function to invert a dictionary (swap keys and values).

2. Implement a simple LRU cache using a dictionary.

3. Write a function to merge multiple dictionaries with conflict resolution.

4. Count word frequencies in a text file.

---

## Next Steps
- [Sets](04_sets.md)
