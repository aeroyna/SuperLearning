# Lists

Lists are ordered, mutable sequences that can contain elements of any type.

---

## 1. Creating Lists

```python
# Empty list
empty = []
empty = list()

# With elements
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True, None]
nested = [[1, 2], [3, 4], [5, 6]]

# From other iterables
list("hello")         # ['h', 'e', 'l', 'l', 'o']
list(range(5))        # [0, 1, 2, 3, 4]
list((1, 2, 3))       # [1, 2, 3]

# List comprehension
squares = [x**2 for x in range(10)]
```

---

## 2. Accessing Elements

### Indexing
```python
lst = ['a', 'b', 'c', 'd', 'e']

lst[0]    # 'a' — first element
lst[1]    # 'b' — second element
lst[-1]   # 'e' — last element
lst[-2]   # 'd' — second to last
```

### Slicing
```python
lst = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

lst[2:5]     # [2, 3, 4] — index 2, 3, 4
lst[:3]      # [0, 1, 2] — first 3 elements
lst[7:]      # [7, 8, 9] — from index 7 to end
lst[::2]     # [0, 2, 4, 6, 8] — every 2nd element
lst[::-1]    # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0] — reversed
lst[1:7:2]   # [1, 3, 5] — index 1 to 7, step 2

# Negative indices
lst[-3:]     # [7, 8, 9] — last 3 elements
lst[:-3]     # [0, 1, 2, 3, 4, 5, 6] — all but last 3
```

---

## 3. Modifying Lists

### Changing Elements
```python
lst = [1, 2, 3, 4, 5]

lst[0] = 10        # [10, 2, 3, 4, 5]
lst[-1] = 50       # [10, 2, 3, 4, 50]
lst[1:3] = [20, 30]  # [10, 20, 30, 4, 50]
```

### Adding Elements
```python
lst = [1, 2, 3]

lst.append(4)         # [1, 2, 3, 4] — add to end
lst.insert(0, 0)      # [0, 1, 2, 3, 4] — add at index
lst.extend([5, 6])    # [0, 1, 2, 3, 4, 5, 6] — add multiple

# Concatenation (creates new list)
new = lst + [7, 8]    # [0, 1, 2, 3, 4, 5, 6, 7, 8]

# Multiplication (creates new list)
[1, 2] * 3            # [1, 2, 1, 2, 1, 2]
```

### Removing Elements
```python
lst = [1, 2, 3, 4, 5, 3]

lst.remove(3)      # [1, 2, 4, 5, 3] — remove first occurrence
lst.pop()          # [1, 2, 4, 5] — remove and return last
lst.pop(0)         # [2, 4, 5] — remove and return at index
del lst[0]         # [4, 5] — delete at index
del lst[0:2]       # [] — delete slice
lst.clear()        # [] — remove all elements
```

---

## 4. List Methods

### Information
```python
lst = [1, 2, 3, 2, 1]

len(lst)           # 5 — length
lst.count(2)       # 2 — count occurrences
lst.index(3)       # 2 — first index of value
# lst.index(99)    # ValueError if not found
```

### Ordering
```python
lst = [3, 1, 4, 1, 5, 9, 2, 6]

lst.sort()                    # Sort in place
lst.sort(reverse=True)        # Sort descending
lst.sort(key=abs)             # Sort by key function

lst.reverse()                 # Reverse in place

# Non-mutating versions
sorted(lst)                   # Returns new sorted list
reversed(lst)                 # Returns iterator
list(reversed(lst))           # Returns new reversed list
```

### Copying
```python
lst = [1, 2, [3, 4]]

# Shallow copy
copy1 = lst.copy()
copy2 = lst[:]
copy3 = list(lst)

# Deep copy (for nested structures)
import copy
deep = copy.deepcopy(lst)
```

---

## 5. List Operations

### Membership Testing
```python
lst = [1, 2, 3, 4, 5]

3 in lst       # True — O(n)
6 not in lst   # True
```

### Concatenation
```python
a = [1, 2]
b = [3, 4]

a + b          # [1, 2, 3, 4] — new list
a += [5, 6]    # a is now [1, 2, 5, 6] — modifies in place
```

### Comparison
```python
[1, 2, 3] == [1, 2, 3]   # True — same elements
[1, 2, 3] < [1, 2, 4]    # True — lexicographic
[1, 2] < [1, 2, 3]       # True — shorter is "less"
```

---

## 6. Iteration

### Basic Loop
```python
for item in lst:
    print(item)
```

### With Index
```python
for i, item in enumerate(lst):
    print(f"{i}: {item}")

# Custom start index
for i, item in enumerate(lst, start=1):
    print(f"{i}: {item}")
```

### Multiple Lists
```python
names = ["Alice", "Bob"]
ages = [25, 30]

for name, age in zip(names, ages):
    print(f"{name} is {age}")
```

---

## 7. Common Patterns

### Filtering
```python
numbers = [1, -2, 3, -4, 5]

# Comprehension
positives = [x for x in numbers if x > 0]

# filter()
positives = list(filter(lambda x: x > 0, numbers))
```

### Mapping
```python
numbers = [1, 2, 3, 4, 5]

# Comprehension
squares = [x**2 for x in numbers]

# map()
squares = list(map(lambda x: x**2, numbers))
```

### Flattening
```python
nested = [[1, 2], [3, 4], [5, 6]]

# Comprehension
flat = [item for sublist in nested for item in sublist]
# [1, 2, 3, 4, 5, 6]

# itertools
from itertools import chain
flat = list(chain.from_iterable(nested))
```

### Finding
```python
lst = [1, 2, 3, 4, 5]

# First match
first_even = next((x for x in lst if x % 2 == 0), None)
# 2

# All matches
evens = [x for x in lst if x % 2 == 0]
# [2, 4]
```

### Aggregation
```python
numbers = [1, 2, 3, 4, 5]

sum(numbers)       # 15
min(numbers)       # 1
max(numbers)       # 5
all(numbers)       # True (all truthy)
any(numbers)       # True (any truthy)
```

---

## 8. List as Stack and Queue

### Stack (LIFO)
```python
stack = []
stack.append(1)    # Push
stack.append(2)
stack.pop()        # Pop → 2
```

### Queue (FIFO)
```python
# Inefficient with list (use collections.deque)
from collections import deque

queue = deque()
queue.append(1)    # Enqueue
queue.append(2)
queue.popleft()    # Dequeue → 1
```

---

## 9. Memory and Performance

### Time Complexity
| Operation | Complexity |
|-----------|------------|
| Index access | O(1) |
| Append | O(1) amortized |
| Insert at beginning | O(n) |
| Delete by index | O(n) |
| Search | O(n) |
| Slice | O(k) where k = slice size |

### Internal Implementation
```python
# Lists over-allocate memory for growth
import sys
lst = []
for i in range(9):
    lst.append(i)
    print(f"Length: {len(lst)}, Size: {sys.getsizeof(lst)}")

# Size grows in chunks, not by 1 each time
```

---

## 10. Practice Problems

1. Write a function to remove duplicates while preserving order.

2. Write a function to rotate a list by k positions.

3. Flatten a deeply nested list of any depth.

4. Find the second largest element in a list.

---

## Next Steps
- [Tuples](02_tuples.md)
