# Variables in Python

## 1. What is a Variable?

In Python, a variable is a **name** that refers to an **object** in memory. Unlike C/C++ where variables are memory locations, Python variables are labels attached to objects.

```python
x = 42
```

This statement:
1. Creates an `int` object with value `42`
2. Binds the name `x` to this object

---

## 2. Naming Rules

### Valid Identifiers
- Must start with a letter (a-z, A-Z) or underscore (_)
- Can contain letters, digits (0-9), and underscores
- Case-sensitive (`name` ≠ `Name` ≠ `NAME`)
- Cannot be a reserved keyword

```python
# Valid
my_var = 1
_private = 2
__dunder__ = 3
CamelCase = 4
variable123 = 5

# Invalid
# 123abc = 1    # Can't start with digit
# my-var = 2    # No hyphens
# class = 3     # Reserved keyword
```

### Python Reserved Keywords
```python
import keyword
print(keyword.kwlist)
# ['False', 'None', 'True', 'and', 'as', 'assert', 'async', 'await',
#  'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except',
#  'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is',
#  'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try',
#  'while', 'with', 'yield']
```

### Naming Conventions (PEP 8)
| Type | Convention | Example |
|------|------------|---------|
| Variables | `snake_case` | `user_name` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_SIZE` |
| Classes | `PascalCase` | `UserAccount` |
| Private | Leading underscore | `_internal` |
| "Private" (name mangling) | Double underscore | `__very_private` |
| Dunder/Magic | Double underscore both | `__init__` |

---

## 3. Variable Assignment

### Basic Assignment
```python
x = 10
name = "Alice"
is_valid = True
```

### Multiple Assignment
```python
# Same value to multiple variables
a = b = c = 0

# Different values (tuple unpacking)
x, y, z = 1, 2, 3

# Swap values
x, y = y, x

# Extended unpacking (Python 3)
first, *rest = [1, 2, 3, 4, 5]
# first = 1, rest = [2, 3, 4, 5]

first, *middle, last = [1, 2, 3, 4, 5]
# first = 1, middle = [2, 3, 4], last = 5
```

### Augmented Assignment
```python
x = 10
x += 5   # x = x + 5 → 15
x -= 3   # x = x - 3 → 12
x *= 2   # x = x * 2 → 24
x /= 4   # x = x / 4 → 6.0
x //= 2  # x = x // 2 → 3.0
x **= 2  # x = x ** 2 → 9.0
x %= 4   # x = x % 4 → 1.0
```

---

## 4. Names, Objects, and References

### The Python Memory Model

```python
x = 300
y = 300
```

Visualization:
```
     ┌─────────────────┐
x ──→│  int object     │
     │  value: 300     │
     │  refcount: 1    │
     └─────────────────┘

     ┌─────────────────┐
y ──→│  int object     │
     │  value: 300     │
     │  refcount: 1    │
     └─────────────────┘
```

### Small Integer Caching
CPython caches integers from -5 to 256:

```python
# Cached range (-5 to 256)
a = 256
b = 256
print(a is b)  # True — same object!

# Outside cache
a = 257
b = 257
print(a is b)  # False — different objects

# But in same expression:
a = 257; b = 257
print(a is b)  # Might be True (compiler optimization)
```

### String Interning
Python automatically interns some strings:

```python
# Interned (simple strings)
a = "hello"
b = "hello"
print(a is b)  # True

# Not interned (has space)
a = "hello world"
b = "hello world"
print(a is b)  # Usually False

# Force interning
import sys
a = sys.intern("hello world")
b = sys.intern("hello world")
print(a is b)  # True
```

---

## 5. Identity, Equality, and Type

### Three Questions About Objects
```python
x = [1, 2, 3]
y = [1, 2, 3]
z = x

# 1. What is it? (type)
type(x)           # <class 'list'>

# 2. What's its identity? (id)
id(x)             # 140234567890 (memory address)
id(z)             # Same as id(x)

# 3. What's its value?
x == y            # True (same value)
x is y            # False (different objects)
x is z            # True (same object)
```

### The `is` Operator vs `==`
```python
# Use == for value comparison
if x == y:
    print("Same value")

# Use 'is' for identity comparison (rare)
if x is None:
    print("x is None")

# Common mistake
if x is [1, 2, 3]:  # BAD! Always False
    pass

if x == [1, 2, 3]:  # Correct
    pass
```

---

## 6. Scope and Namespaces

### LEGB Rule
Python looks up names in this order:
1. **L**ocal — Inside the current function
2. **E**nclosing — In enclosing functions (closures)
3. **G**lobal — Module level
4. **B**uilt-in — Python's built-in names

```python
x = "global"  # Global

def outer():
    x = "enclosing"  # Enclosing

    def inner():
        x = "local"  # Local
        print(x)     # "local"

    inner()
    print(x)         # "enclosing"

outer()
print(x)             # "global"
```

### The `global` Keyword
```python
counter = 0

def increment():
    global counter
    counter += 1

increment()
print(counter)  # 1
```

### The `nonlocal` Keyword
```python
def outer():
    x = 0

    def inner():
        nonlocal x
        x += 1

    inner()
    print(x)  # 1

outer()
```

---

## 7. Deleting Variables

```python
x = 10
del x
# print(x)  # NameError: name 'x' is not defined

# Delete from a list
lst = [1, 2, 3]
del lst[1]
print(lst)  # [1, 3]

# Delete from a dict
d = {"a": 1, "b": 2}
del d["a"]
print(d)  # {"b": 2}
```

---

## 8. Best Practices

### Do's
```python
# Use descriptive names
user_count = 42  # ✓

# Use constants for magic numbers
MAX_RETRIES = 3
for i in range(MAX_RETRIES):
    pass

# Unpack sensibly
x, y = point
first, *rest = items
```

### Don'ts
```python
# Avoid single letters (except in specific contexts)
# n = 42  # ✗ (unless it's a loop counter or math)

# Avoid shadowing built-ins
# list = [1, 2, 3]  # ✗ shadows built-in list()
# id = 42           # ✗ shadows built-in id()

# Avoid misleading names
# is_valid = 1      # ✗ looks like bool, is int
```

---

## 9. Practice Problems

1. What's the output?
```python
a = [1, 2, 3]
b = a
a = [4, 5, 6]
print(b)
```

2. What's the output?
```python
a = [1, 2, 3]
b = a
a.append(4)
print(b)
```

3. What's the output?
```python
x = 256
y = 256
print(x is y)

x = 257
y = 257
print(x is y)
```

---

## Next Steps
- [Numeric Types](02_numeric_types.md)
