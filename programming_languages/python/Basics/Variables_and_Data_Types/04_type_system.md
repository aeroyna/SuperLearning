# Python's Type System

Understanding Python's type system is crucial for writing robust code and debugging effectively.

---

## 1. Dynamic Typing

### What is Dynamic Typing?
In Python, **types are associated with objects, not variables**. Variables are just names that can reference any object.

```python
x = 42        # x references an int
x = "hello"   # x now references a str
x = [1, 2, 3] # x now references a list

# The TYPE is attached to the object, not the variable
```

### Compared to Static Typing (C++/Java)
```cpp
// C++ — variable has fixed type
int x = 42;
x = "hello";  // Compile error!
```

```python
# Python — variable is just a name
x = 42
x = "hello"  # Fine!
```

### Runtime Type Checking
```python
def add(a, b):
    return a + b

add(1, 2)        # 3
add("a", "b")    # "ab"
add(1, "2")      # TypeError at runtime!
```

---

## 2. Strong Typing

Python is **strongly typed** — it doesn't implicitly convert between unrelated types:

```python
# No implicit conversion
"hello" + 42    # TypeError!
"hello" + str(42)  # "hello42" — explicit conversion

# Contrast with JavaScript (weakly typed)
# "hello" + 42 → "hello42" (implicit conversion)
```

### Type Coercion Rules
Python does some numeric coercion:

```python
# Numeric tower: complex > float > int > bool
1 + 1.0        # 2.0 (int + float → float)
1 + True       # 2 (int + bool → int, True=1)
1.0 + (2+3j)   # (3+3j) (float + complex → complex)
```

---

## 3. Duck Typing

### "If it walks like a duck..."
Python focuses on **behavior**, not declared types:

```python
def get_length(obj):
    return len(obj)  # Works if obj has __len__

get_length([1, 2, 3])      # 3 — list has __len__
get_length("hello")        # 5 — str has __len__
get_length({1: 'a', 2: 'b'})  # 2 — dict has __len__
```

### Protocols Over Types
```python
# Iterable protocol: anything with __iter__
def print_all(iterable):
    for item in iterable:
        print(item)

print_all([1, 2, 3])       # List
print_all((1, 2, 3))       # Tuple
print_all({1, 2, 3})       # Set
print_all("abc")           # String
print_all(range(3))        # Range
```

### EAFP vs LBYL

**EAFP** (Easier to Ask Forgiveness than Permission) — Pythonic:
```python
def get_length(obj):
    try:
        return len(obj)
    except TypeError:
        return None
```

**LBYL** (Look Before You Leap) — Less Pythonic:
```python
def get_length(obj):
    if hasattr(obj, '__len__'):
        return len(obj)
    return None
```

---

## 4. Type Introspection

### type() Function
```python
type(42)          # <class 'int'>
type("hello")     # <class 'str'>
type([1, 2, 3])   # <class 'list'>
type(print)       # <class 'builtin_function_or_method'>

# type() returns the type object
type(42) is int   # True
```

### isinstance() Function
```python
# Check if object is instance of type (or subtype)
isinstance(42, int)        # True
isinstance(True, int)      # True — bool is subclass of int
isinstance(42, (int, str)) # True — check multiple types

# Preferred over type() for type checking
# because it respects inheritance
```

### issubclass() Function
```python
issubclass(bool, int)      # True
issubclass(int, object)    # True
issubclass(str, (int, bytes))  # False
```

### __class__ Attribute
```python
x = 42
x.__class__        # <class 'int'>
x.__class__.__name__  # 'int'
```

---

## 5. Type Hierarchy

### Object is the Root
```python
# Everything inherits from object
isinstance(42, object)       # True
isinstance("hi", object)     # True
isinstance(None, object)     # True
isinstance(type, object)     # True

# Even type itself!
isinstance(object, type)     # True
isinstance(type, object)     # True
```

### The Type/Object Relationship
```
          object ←──── Everything inherits from object
            ↑
           type ←───── type is the metaclass of all classes
            ↑
          class ←───── User-defined classes
```

```python
type(42)        # <class 'int'> — 42 is instance of int
type(int)       # <class 'type'> — int is instance of type
type(type)      # <class 'type'> — type is instance of type
type(object)    # <class 'type'> — object is instance of type
```

---

## 6. Mutable vs Immutable Types

### Immutable Types
Objects that cannot be changed after creation:

```python
# Immutable: int, float, str, tuple, frozenset, bytes

x = 5
y = x
x += 1
print(y)  # Still 5 — new int object created

s = "hello"
# s[0] = "H"  # TypeError! Strings are immutable
s = "H" + s[1:]  # Create new string instead
```

### Mutable Types
Objects that can be modified in place:

```python
# Mutable: list, dict, set, bytearray

lst = [1, 2, 3]
lst2 = lst
lst.append(4)
print(lst2)  # [1, 2, 3, 4] — same object modified!
```

### Why Does This Matter?

```python
# Function arguments
def modify(lst):
    lst.append(4)  # Modifies original!

my_list = [1, 2, 3]
modify(my_list)
print(my_list)  # [1, 2, 3, 4]

# Dictionary keys must be hashable (immutable)
d = {}
d[[1, 2]] = "value"  # TypeError! Lists are unhashable
d[(1, 2)] = "value"  # OK — tuples are hashable
```

---

## 7. Type Annotations (Preview)

Python 3.5+ supports **optional type hints**:

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

def add_numbers(a: int, b: int) -> int:
    return a + b

# Type hints are NOT enforced at runtime!
greet(42)  # Works (no error), returns "Hello, 42"
```

Type hints enable:
- Better IDE support (autocomplete, error detection)
- Static type checking with tools like `mypy`
- Self-documenting code

```python
from typing import List, Dict, Optional

def process_items(items: List[str]) -> Dict[str, int]:
    return {item: len(item) for item in items}

def find_user(user_id: int) -> Optional[str]:
    # Returns str or None
    users = {1: "Alice", 2: "Bob"}
    return users.get(user_id)
```

See [Type Hints and Static Typing](../../Modern_Python/Type_Hints/00_type_hints.md) for comprehensive coverage.

---

## 8. Common Type Operations

### Type Conversion
```python
# Built-in conversion functions
int("42")       # 42
float("3.14")   # 3.14
str(42)         # "42"
list("abc")     # ['a', 'b', 'c']
tuple([1,2,3])  # (1, 2, 3)
set([1,1,2,2])  # {1, 2}
dict([('a',1), ('b',2)])  # {'a': 1, 'b': 2}
bool(0)         # False
```

### Type-Specific Checks
```python
# String checks
"123".isdigit()    # True
"abc".isalpha()    # True
"abc123".isalnum() # True

# Number checks
import math
math.isnan(float('nan'))  # True
math.isinf(float('inf'))  # True
(4.0).is_integer()        # True
```

---

## 9. Practice Problems

1. What's the output?
```python
print(type(type(42)))
print(isinstance(True, int))
print(issubclass(bool, object))
```

2. Why does this happen?
```python
a = 256
b = 256
print(a is b)  # ?

c = 257
d = 257
print(c is d)  # ?
```

3. What's the difference?
```python
def func1(x):
    if type(x) == list:
        return len(x)

def func2(x):
    if isinstance(x, list):
        return len(x)
```

---

## Next Steps
- [Practice Problems](05_practice_problems.md)
