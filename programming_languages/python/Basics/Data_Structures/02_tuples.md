# Tuples

Tuples are ordered, immutable sequences, often used for fixed collections of heterogeneous data.

---

## 1. Creating Tuples

```python
# Empty tuple
empty = ()
empty = tuple()

# With elements
point = (3, 4)
rgb = (255, 128, 64)
mixed = (1, "hello", 3.14, True)
nested = ((1, 2), (3, 4))

# Without parentheses (tuple packing)
coordinates = 3, 4, 5

# Single element tuple (comma required!)
single = (42,)    # Correct
not_tuple = (42)  # Just the integer 42

# From other iterables
tuple([1, 2, 3])      # (1, 2, 3)
tuple("hello")        # ('h', 'e', 'l', 'l', 'o')
tuple(range(5))       # (0, 1, 2, 3, 4)
```

---

## 2. Accessing Elements

```python
point = (3, 4, 5)

# Indexing
point[0]      # 3
point[-1]     # 5

# Slicing (returns new tuple)
point[0:2]    # (3, 4)
point[::-1]   # (5, 4, 3)
```

---

## 3. Immutability

Tuples cannot be modified after creation:

```python
t = (1, 2, 3)

# Cannot change elements
# t[0] = 10  # TypeError: 'tuple' object does not support item assignment

# Cannot add elements
# t.append(4)  # AttributeError: 'tuple' object has no attribute 'append'

# But: mutable elements inside can be changed!
t = ([1, 2], [3, 4])
t[0].append(5)  # OK! The list inside can be modified
print(t)  # ([1, 2, 5], [3, 4])
```

---

## 4. Tuple Unpacking

### Basic Unpacking
```python
point = (3, 4)
x, y = point
print(x, y)  # 3 4

# Swap values
a, b = 1, 2
a, b = b, a
print(a, b)  # 2 1
```

### Extended Unpacking
```python
first, *rest = (1, 2, 3, 4, 5)
# first = 1, rest = [2, 3, 4, 5]

first, *middle, last = (1, 2, 3, 4, 5)
# first = 1, middle = [2, 3, 4], last = 5

*start, last = (1, 2, 3, 4, 5)
# start = [1, 2, 3, 4], last = 5
```

### Ignoring Values
```python
x, _, z = (1, 2, 3)  # Ignore second value

first, *_ = (1, 2, 3, 4, 5)  # Only need first
```

### Function Returns
```python
def min_max(numbers):
    return min(numbers), max(numbers)

minimum, maximum = min_max([1, 5, 3, 9, 2])
```

---

## 5. Named Tuples

For more readable code, use `namedtuple`:

```python
from collections import namedtuple

# Define a named tuple type
Point = namedtuple('Point', ['x', 'y'])

# Create instances
p = Point(3, 4)
p = Point(x=3, y=4)

# Access by name or index
p.x        # 3
p.y        # 4
p[0]       # 3
p[1]       # 4

# Still a tuple
isinstance(p, tuple)  # True
x, y = p              # Unpacking works
```

### Named Tuple with Defaults (Python 3.7+)
```python
Point = namedtuple('Point', ['x', 'y', 'z'], defaults=[0])
Point(1, 2)      # Point(x=1, y=2, z=0)
Point(1, 2, 3)   # Point(x=1, y=2, z=3)
```

### Named Tuple Methods
```python
Point = namedtuple('Point', ['x', 'y'])
p = Point(3, 4)

p._asdict()      # {'x': 3, 'y': 4}
p._replace(x=5)  # Point(x=5, y=4) — returns new tuple
p._fields        # ('x', 'y')

# Create from dict
Point(**{'x': 3, 'y': 4})  # Point(x=3, y=4)
```

---

## 6. Tuple Methods

Tuples have only two methods:

```python
t = (1, 2, 3, 2, 1)

t.count(2)    # 2 — count occurrences
t.index(3)    # 2 — first index of value
```

---

## 7. Tuple Operations

### Concatenation
```python
(1, 2) + (3, 4)      # (1, 2, 3, 4)
```

### Repetition
```python
(1, 2) * 3           # (1, 2, 1, 2, 1, 2)
```

### Membership
```python
3 in (1, 2, 3, 4)    # True
```

### Comparison
```python
(1, 2, 3) == (1, 2, 3)   # True
(1, 2, 3) < (1, 2, 4)    # True — lexicographic
```

---

## 8. Tuples as Dictionary Keys

Because tuples are immutable (and hashable if all elements are hashable), they can be dictionary keys:

```python
# Coordinate -> value mapping
grid = {
    (0, 0): "origin",
    (1, 0): "right",
    (0, 1): "up",
}

grid[(0, 0)]  # "origin"

# Multi-key dictionary
cache = {}
cache[(func_name, arg1, arg2)] = result
```

---

## 9. Tuple vs List

### Use Tuple When:
- Data is fixed and shouldn't change
- Using as dictionary key
- Returning multiple values from function
- Passing data that shouldn't be modified
- Performance matters (slightly faster)

### Use List When:
- Data needs to be modified
- Collection grows/shrinks
- Order might change

```python
# Tuple: fixed structure
person = ("Alice", 30, "Engineer")

# List: dynamic collection
tasks = ["buy milk", "write code"]
tasks.append("review PR")
```

---

## 10. Performance

```python
import sys

# Tuples use less memory
sys.getsizeof((1, 2, 3))  # 64 bytes
sys.getsizeof([1, 2, 3])  # 88 bytes

# Tuples are slightly faster to create
import timeit
timeit.timeit("(1, 2, 3)")     # ~0.01 μs
timeit.timeit("[1, 2, 3]")     # ~0.05 μs
```

---

## 11. Common Patterns

### Multiple Return Values
```python
def analyze(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

minimum, maximum, average = analyze([1, 2, 3, 4, 5])
```

### Coordinate/Point Representation
```python
origin = (0, 0)
destination = (100, 200)

def distance(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return ((x2 - x1)**2 + (y2 - y1)**2) ** 0.5
```

### Record-like Data
```python
# Before named tuples
employee = ("Alice", "Engineering", 50000)
name, dept, salary = employee

# With named tuple (better)
Employee = namedtuple('Employee', ['name', 'dept', 'salary'])
employee = Employee("Alice", "Engineering", 50000)
print(employee.name)  # "Alice"
```

### Argument Unpacking
```python
args = (1, 2, 3)
func(*args)  # Same as func(1, 2, 3)
```

---

## 12. Practice Problems

1. Write a function that returns the two largest numbers in a list as a tuple.

2. Create a named tuple `RGB` and write a function to blend two colors.

3. Write a function that takes a list of (x, y) tuples and returns the one closest to the origin.

---

## Next Steps
- [Dictionaries](03_dictionaries.md)
