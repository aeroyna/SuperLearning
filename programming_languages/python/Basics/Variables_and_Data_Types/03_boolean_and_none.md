# Boolean and None Types

## 1. Boolean Type (`bool`)

### The Two Boolean Values
```python
True   # Boolean true
False  # Boolean false

type(True)   # <class 'bool'>
type(False)  # <class 'bool'>
```

### Bool is a Subclass of Int
```python
# Boolean inherits from int
isinstance(True, int)   # True
isinstance(False, int)  # True

# True = 1, False = 0
True + True   # 2
False + True  # 1
True * 10     # 10

# But comparison shows bool
type(True + False)  # <class 'int'> — arithmetic returns int
```

---

## 2. Truthiness and Falsiness

### Falsy Values
These evaluate to `False` in a boolean context:

```python
bool(None)      # False
bool(False)     # False
bool(0)         # False
bool(0.0)       # False
bool(0j)        # False
bool('')        # False — empty string
bool([])        # False — empty list
bool(())        # False — empty tuple
bool({})        # False — empty dict
bool(set())     # False — empty set
```

### Truthy Values
Everything else is truthy:

```python
bool(1)         # True
bool(-1)        # True — any non-zero number
bool(0.0001)    # True
bool('hello')   # True — non-empty string
bool([0])       # True — non-empty list (even with falsy element)
bool(' ')       # True — space is not empty!
```

### Custom Truthiness
Objects can define their own truthiness with `__bool__` or `__len__`:

```python
class MyClass:
    def __init__(self, value):
        self.value = value

    def __bool__(self):
        return self.value > 0

obj1 = MyClass(5)
obj2 = MyClass(-1)

bool(obj1)  # True
bool(obj2)  # False

if obj1:
    print("obj1 is truthy")
```

---

## 3. Boolean Operations

### Logical Operators
```python
# and — returns first falsy value or last value
True and True    # True
True and False   # False
False and True   # False
0 and 1          # 0 (first falsy)
1 and 2          # 2 (last value if all truthy)
'' and 'hello'   # '' (first falsy)

# or — returns first truthy value or last value
True or False    # True
False or True    # True
False or False   # False
0 or 1           # 1 (first truthy)
'' or 'default'  # 'default' (first truthy)
None or 0 or ''  # '' (last value if all falsy)

# not — returns boolean
not True   # False
not False  # True
not 0      # True
not 1      # False
not []     # True
not 'hi'   # False
```

### Short-Circuit Evaluation
```python
# and stops at first falsy
def side_effect():
    print("called!")
    return True

False and side_effect()  # side_effect() never called

# or stops at first truthy
True or side_effect()    # side_effect() never called
```

### Common Patterns
```python
# Default value with or
name = user_input or "Anonymous"

# Guard with and
result = data and data.process()  # Only calls process() if data is truthy

# Ternary with and/or (avoid — use if-else)
# value = condition and true_val or false_val  # WRONG if true_val is falsy
value = true_val if condition else false_val   # CORRECT
```

---

## 4. Comparison Operators

### Value Comparison
```python
# Equality
5 == 5      # True
5 != 3      # True
"a" == "a"  # True

# Ordering
5 < 10      # True
5 <= 5      # True
10 > 5      # True
10 >= 10    # True

# Chained comparison
1 < 5 < 10  # True (equivalent to: 1 < 5 and 5 < 10)
1 < 5 > 3   # True (1 < 5 and 5 > 3)
```

### Identity Comparison
```python
# is / is not — checks object identity
a = [1, 2]
b = [1, 2]
c = a

a == b   # True (same value)
a is b   # False (different objects)
a is c   # True (same object)
```

### Membership Testing
```python
# in / not in
'a' in 'abc'        # True
1 in [1, 2, 3]      # True
'key' in {'key': 1} # True (checks keys)
5 not in range(3)   # True
```

---

## 5. The None Type

### What is None?
`None` is Python's null value — it represents the absence of a value.

```python
x = None
type(x)  # <class 'NoneType'>
```

### Singleton Pattern
`None` is a singleton — there's only one `None` object:

```python
a = None
b = None
a is b   # True — always use 'is' to check for None
```

### Common Uses of None

#### 1. Default Return Value
```python
def no_return():
    x = 1 + 1
    # No explicit return

result = no_return()
print(result)  # None
```

#### 2. Default Function Arguments
```python
# WRONG — mutable default
def append_to(item, lst=[]):
    lst.append(item)
    return lst

append_to(1)  # [1]
append_to(2)  # [1, 2] — oops! Same list

# CORRECT — use None
def append_to(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst

append_to(1)  # [1]
append_to(2)  # [2] — new list each time
```

#### 3. Optional Values
```python
def find_user(user_id):
    # Returns User object or None if not found
    user = database.get(user_id)
    return user  # might be None

user = find_user(123)
if user is not None:
    print(user.name)
```

#### 4. Sentinel Value
```python
_MISSING = object()  # Unique sentinel

def get_value(d, key, default=_MISSING):
    if key in d:
        return d[key]
    if default is _MISSING:
        raise KeyError(key)
    return default
```

### Checking for None
```python
x = None

# CORRECT — use identity
if x is None:
    print("x is None")

if x is not None:
    print("x has a value")

# AVOID — equality works but is not idiomatic
if x == None:   # Works, but use 'is' instead
    pass

# DANGER — empty values are also falsy!
if not x:      # True for None, but also for 0, '', [], etc.
    pass
```

---

## 6. Boolean Context in Control Flow

### If Statements
```python
# Explicit comparison (sometimes needed)
if count == 0:
    print("empty")

# Implicit truthiness (Pythonic)
if items:  # True if non-empty
    print("has items")

if not errors:  # True if empty
    print("no errors")
```

### While Loops
```python
# Loop until empty
stack = [1, 2, 3]
while stack:
    item = stack.pop()
    print(item)
```

### Filter and Comprehensions
```python
# Filter falsy values
items = [0, 1, '', 'hello', None, [], [1]]
truthy = [x for x in items if x]
# [1, 'hello', [1]]

# Using filter
truthy = list(filter(None, items))  # Same result
```

---

## 7. The bool() Function

```python
# Convert to boolean
bool(42)      # True
bool(0)       # False
bool("yes")   # True
bool("")      # False

# Useful for explicit conversion
print(bool([]))  # False — clearer than just []
```

---

## 8. Common Pitfalls

### 1. Comparing to True/False
```python
# BAD
if is_valid == True:
    pass

# GOOD
if is_valid:
    pass

# BAD
if is_empty == False:
    pass

# GOOD
if not is_empty:
    pass
```

### 2. Confusing `is` and `==` with Booleans
```python
# These work but are unnecessary
if x is True:   # Too specific — what if x is 1?
    pass

if x == True:   # Works but verbose
    pass

# Better
if x:
    pass
```

### 3. None vs Empty
```python
def process(items):
    # None means "not provided"
    # [] means "explicitly empty"

    if items is None:
        items = get_default_items()

    if not items:  # Now safe to check emptiness
        return "Nothing to process"
```

---

## 9. Practice Problems

1. What's the output?
```python
print(bool('False'))
print(bool([None]))
print(bool(0.0))
```

2. What does this return?
```python
x = None or [] or 0 or "hello" or False
```

3. Fix this function:
```python
def add_item(item, items=[]):
    items.append(item)
    return items
```

---

## Next Steps
- [Type System](04_type_system.md)
