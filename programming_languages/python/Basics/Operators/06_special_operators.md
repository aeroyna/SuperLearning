# Special Operators

## 1. Identity Operators

### `is` and `is not`

Compare object **identity** (memory address):

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

a == b     # True (same value)
a is b     # False (different objects)
a is c     # True (same object)
a is not b # True
```

### When to Use `is`

```python
# Always use 'is' for None
if x is None:
    pass

if x is not None:
    pass

# Use 'is' for sentinel values
MISSING = object()
if value is MISSING:
    pass

# Use 'is' for True/False (rare)
if flag is True:  # Usually just: if flag:
    pass
```

### Pitfalls

```python
# Small integers are cached (-5 to 256)
a = 256
b = 256
a is b  # True — same cached object

a = 257
b = 257
a is b  # False — different objects (usually)

# Empty collections
[] is []  # False — different objects
() is ()  # True — empty tuple is singleton
```

---

## 2. Membership Operators

### `in` and `not in`

Test if a value is in a container:

```python
# Lists
3 in [1, 2, 3]        # True
5 in [1, 2, 3]        # False
5 not in [1, 2, 3]    # True

# Strings
'a' in 'abc'          # True
'ab' in 'abc'         # True (substring)
'ac' in 'abc'         # False

# Tuples
2 in (1, 2, 3)        # True

# Sets (O(1) lookup)
2 in {1, 2, 3}        # True

# Dictionaries (checks keys, not values)
'a' in {'a': 1, 'b': 2}  # True
1 in {'a': 1, 'b': 2}    # False (1 is a value, not key)

# Range (O(1) in Python 3)
50 in range(100)      # True
```

### Custom `__contains__`

```python
class Range:
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def __contains__(self, item):
        return self.start <= item < self.end

r = Range(1, 10)
5 in r    # True
15 in r   # False
```

---

## 3. Index and Slice Operators

### Indexing `[]`

```python
lst = [10, 20, 30, 40, 50]

lst[0]    # 10 (first element)
lst[-1]   # 50 (last element)
lst[-2]   # 40 (second to last)
```

### Slicing `[start:stop:step]`

```python
lst = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

lst[2:5]     # [2, 3, 4] (index 2, 3, 4)
lst[:3]      # [0, 1, 2] (first 3)
lst[7:]      # [7, 8, 9] (from index 7 to end)
lst[::2]     # [0, 2, 4, 6, 8] (every 2nd element)
lst[::-1]    # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0] (reversed)
lst[1:7:2]   # [1, 3, 5] (1 to 7, step 2)

# Negative indices
lst[-3:]     # [7, 8, 9] (last 3)
lst[:-3]     # [0, 1, 2, 3, 4, 5, 6] (all but last 3)
lst[-5:-2]   # [5, 6, 7]
```

### Slice Assignment

```python
lst = [0, 1, 2, 3, 4]

lst[1:3] = [10, 20, 30]  # Replace slice
# lst = [0, 10, 20, 30, 3, 4]

lst[1:4] = []  # Delete elements
# lst = [0, 3, 4]

lst[1:1] = [1, 2]  # Insert elements
# lst = [0, 1, 2, 3, 4]
```

---

## 4. Call Operator `()`

Makes objects callable:

```python
# Functions
def greet(name):
    return f"Hello, {name}!"

greet("World")  # "Hello, World!"

# Classes (calls __call__)
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, value):
        return value * self.factor

double = Multiplier(2)
double(5)  # 10
```

---

## 5. Attribute Access Operators

### `.` Operator

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
p.x  # 3
p.y  # 4
```

### `getattr`, `setattr`, `delattr`, `hasattr`

```python
class Obj:
    x = 10

o = Obj()

# Get attribute
getattr(o, 'x')           # 10
getattr(o, 'y', 'default')  # 'default'

# Set attribute
setattr(o, 'y', 20)
o.y  # 20

# Check attribute
hasattr(o, 'x')  # True
hasattr(o, 'z')  # False

# Delete attribute
delattr(o, 'y')
# o.y would raise AttributeError
```

---

## 6. Ternary Conditional Operator

```python
# syntax: value_if_true if condition else value_if_false

x = 5
result = "positive" if x > 0 else "non-positive"
# result = "positive"

# Can be chained (but avoid deep nesting)
result = "positive" if x > 0 else "zero" if x == 0 else "negative"

# In expressions
print(f"x is {'even' if x % 2 == 0 else 'odd'}")
```

---

## 7. Unpacking Operators

### `*` for Iterables

```python
# In function calls
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
add(*nums)  # 6 (same as add(1, 2, 3))

# In literals (Python 3.5+)
a = [1, 2]
b = [3, 4]
combined = [*a, *b]  # [1, 2, 3, 4]

# With sets
s1 = {1, 2}
s2 = {2, 3}
{*s1, *s2}  # {1, 2, 3}
```

### `**` for Dictionaries

```python
# In function calls
def greet(first, last):
    return f"Hello, {first} {last}!"

person = {'first': 'John', 'last': 'Doe'}
greet(**person)  # "Hello, John Doe!"

# In literals (Python 3.5+)
d1 = {'a': 1, 'b': 2}
d2 = {'c': 3, 'd': 4}
combined = {**d1, **d2}  # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# Override values
d1 = {'a': 1, 'b': 2}
d2 = {'b': 20, 'c': 3}
{**d1, **d2}  # {'a': 1, 'b': 20, 'c': 3}
```

---

## 8. Matrix Multiplication Operator `@` (Python 3.5+)

```python
# Used with NumPy and similar libraries
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

A @ B  # Matrix multiplication
# array([[19, 22],
#        [43, 50]])

# Equivalent to
np.matmul(A, B)
```

---

## 9. Dictionary Merge Operators (Python 3.9+)

### `|` for Merging

```python
d1 = {'a': 1, 'b': 2}
d2 = {'b': 20, 'c': 3}

# Merge (right side wins on conflict)
d1 | d2  # {'a': 1, 'b': 20, 'c': 3}
d2 | d1  # {'b': 2, 'c': 3, 'a': 1}

# Original dicts unchanged
```

### `|=` for Update

```python
d1 = {'a': 1, 'b': 2}
d1 |= {'c': 3}
# d1 = {'a': 1, 'b': 2, 'c': 3}
```

---

## 10. Practice Problems

1. What's the output?
```python
a = 256
b = 256
c = 257
d = 257
print(a is b, c is d)
```

2. Check if a string contains any vowels using `in`:
```python
def has_vowel(s):
    # Your code
    pass
```

3. What does this return?
```python
d1 = {'x': 1}
d2 = {'x': 2, 'y': 3}
result = {**d1, **d2, 'z': 4}
```

---

## Next Steps
- [Operator Overloading](07_operator_overloading.md)
