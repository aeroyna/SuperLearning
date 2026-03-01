# Variables and Data Types

Python's type system is one of its most distinctive features. Understanding how Python handles variables and types is fundamental to writing effective code.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Variables**](01_variables.md) | Names, binding, and memory model |
| [**2. Numeric Types**](02_numeric_types.md) | int, float, complex, and their internals |
| [**3. Boolean and None**](03_boolean_and_none.md) | Truth values and null representation |
| [**4. Type System**](04_type_system.md) | Dynamic typing, duck typing, and type introspection |
| [**5. Practice Problems**](05_practice_problems.md) | Exercises to test your understanding |

---

## Key Concepts

### Dynamic Typing
Python is **dynamically typed**—variables don't have fixed types:

```python
x = 42        # x is an int
x = "hello"   # now x is a str
x = [1, 2, 3] # now x is a list
```

### Strong Typing
Python is **strongly typed**—implicit type coercion is limited:

```python
"hello" + 42  # TypeError! Python won't implicitly convert
"hello" + str(42)  # Works: "hello42"
```

### Everything is an Object
In Python, **everything is an object**, including functions and classes:

```python
>>> type(42)
<class 'int'>
>>> type("hello")
<class 'str'>
>>> type(print)
<class 'builtin_function_or_method'>
>>> type(int)
<class 'type'>
```

---

## Built-in Types Overview

| Category | Types |
|----------|-------|
| **Numeric** | `int`, `float`, `complex`, `bool` |
| **Sequence** | `str`, `list`, `tuple`, `range`, `bytes`, `bytearray` |
| **Mapping** | `dict` |
| **Set** | `set`, `frozenset` |
| **None** | `NoneType` |
| **Callable** | Functions, methods, classes |

---

## Memory Model: Names and Objects

Python variables are **names** that reference **objects**:

```python
a = [1, 2, 3]  # Create list object, bind name 'a' to it
b = a          # Bind name 'b' to the SAME object
b.append(4)
print(a)       # [1, 2, 3, 4] — both names refer to same object!
```

Visualized:
```
Names          Objects
  a ──────────→ [1, 2, 3, 4]
  b ──────────↗
```

### Identity vs Equality
```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

# Equality (same value)
a == b  # True

# Identity (same object)
a is b  # False (different objects)
a is c  # True (same object)

# Check identity with id()
id(a)  # e.g., 140234567890
id(b)  # e.g., 140234567456 (different)
id(c)  # Same as id(a)
```

---

## Mutability

| Mutable | Immutable |
|---------|-----------|
| `list`, `dict`, `set`, `bytearray` | `int`, `float`, `str`, `tuple`, `frozenset`, `bytes` |

This distinction is crucial for understanding Python behavior:

```python
# Immutable: creates new object
x = 5
x += 1  # x now points to a NEW int object (6)

# Mutable: modifies in place
lst = [1, 2, 3]
lst.append(4)  # Same list object, modified
```

---

## Next Steps
Start with [Variables](01_variables.md) to understand Python's naming and binding system.
