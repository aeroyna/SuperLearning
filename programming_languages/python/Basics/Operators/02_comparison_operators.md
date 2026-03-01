# Comparison Operators

## 1. Basic Comparisons

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `==` | Equal | `5 == 5` | `True` |
| `!=` | Not equal | `5 != 3` | `True` |
| `<` | Less than | `3 < 5` | `True` |
| `>` | Greater than | `5 > 3` | `True` |
| `<=` | Less or equal | `5 <= 5` | `True` |
| `>=` | Greater or equal | `5 >= 3` | `True` |

---

## 2. Equality vs Identity

### `==` (Equality)
Compares **values**:

```python
a = [1, 2, 3]
b = [1, 2, 3]

a == b  # True — same value
```

### `is` (Identity)
Compares **object identity** (memory address):

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

a is b  # False — different objects
a is c  # True — same object
```

### When to Use Each
```python
# Use == for value comparison
if user_input == "yes":
    pass

# Use 'is' for None, True, False (singletons)
if result is None:
    pass

if flag is True:  # Usually just: if flag:
    pass
```

---

## 3. Chained Comparisons

Python allows chaining comparisons:

```python
# Range check
1 < 5 < 10  # True — equivalent to: (1 < 5) and (5 < 10)

# Multiple conditions
x = 5
1 <= x <= 10  # True

# Mixed operators
1 < 2 < 3 < 4  # True
1 < 2 > 0     # True (1 < 2) and (2 > 0)
```

### How Chaining Works
```python
# a < b < c is equivalent to:
# (a < b) and (b < c)
# BUT b is only evaluated once!

def check():
    print("called!")
    return 5

1 < check() < 10  # prints "called!" once, returns True
```

---

## 4. Comparing Different Types

### Numeric Types
Numbers of different types can be compared:

```python
5 == 5.0     # True
5 == 5+0j    # True
True == 1    # True
False == 0   # True
```

### Strings
Lexicographic comparison (Unicode code points):

```python
"apple" < "banana"  # True
"Apple" < "apple"   # True (uppercase < lowercase)
"10" < "9"          # True (string comparison!)

# Compare strings properly
"10" < "9"     # True — wrong for numbers!
int("10") < int("9")  # False — correct
```

### Sequences
Element-by-element comparison:

```python
[1, 2, 3] < [1, 2, 4]     # True
[1, 2] < [1, 2, 3]        # True (shorter)
(1, 2, 3) == (1, 2, 3)    # True
[1, 2, 3] == (1, 2, 3)    # False (different types!)
```

### Incompatible Types
```python
5 < "5"  # TypeError in Python 3!
# Python 2 would allow this (wrong)
```

---

## 5. Object Comparison

### Default Behavior
Objects compare by identity by default:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p1 = Point(1, 2)
p2 = Point(1, 2)

p1 == p2  # False — different objects!
```

### Custom Comparison with `__eq__`
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)

p1 == p2  # True
```

### Rich Comparison Methods
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other): return (self.x, self.y) == (other.x, other.y)
    def __ne__(self, other): return (self.x, self.y) != (other.x, other.y)
    def __lt__(self, other): return (self.x, self.y) < (other.x, other.y)
    def __le__(self, other): return (self.x, self.y) <= (other.x, other.y)
    def __gt__(self, other): return (self.x, self.y) > (other.x, other.y)
    def __ge__(self, other): return (self.x, self.y) >= (other.x, other.y)

# Or use functools.total_ordering
from functools import total_ordering

@total_ordering
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)

    def __lt__(self, other):
        return (self.x, self.y) < (other.x, other.y)
    # Other comparison methods are auto-generated!
```

---

## 6. Special Cases

### NaN Comparisons
```python
import math

nan = float('nan')

nan == nan   # False! NaN is not equal to itself
nan != nan   # True
nan < 0      # False
nan > 0      # False

# Use math.isnan() instead
math.isnan(nan)  # True
```

### None Comparisons
```python
None == None  # True
None is None  # True (preferred)

# Can't compare None with numbers
None < 0   # TypeError
None == 0  # False (but no error)
```

### Boolean Comparisons
```python
# Booleans are ints
True == 1   # True
False == 0  # True
True > False  # True

# But prefer direct boolean checks
if flag:       # Better than: if flag == True:
    pass

if not flag:   # Better than: if flag == False:
    pass
```

---

## 7. Practice Problems

1. What's the output?
```python
print([1, 2] == [1, 2])
print([1, 2] is [1, 2])
print("abc" < "abd")
print((1, 2, 3) < (1, 2, 4))
```

2. Is this True or False?
```python
x = float('nan')
x == x or x != x
```

3. Write a function that checks if three values are in ascending order:
```python
def is_ascending(a, b, c):
    # Your code
    pass

is_ascending(1, 2, 3)  # True
is_ascending(1, 3, 2)  # False
```

---

## Next Steps
- [Logical Operators](03_logical_operators.md)
