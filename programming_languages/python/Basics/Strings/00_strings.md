# Strings

Strings are immutable sequences of Unicode characters, one of Python's most commonly used data types.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. String Basics**](01_string_basics.md) | Creation, indexing, slicing |
| [**2. String Methods**](02_string_methods.md) | Manipulation and transformation |
| [**3. String Formatting**](03_string_formatting.md) | f-strings, format(), % operator |

---

## Quick Reference

### Creating Strings
```python
s = "Hello"           # Double quotes
s = 'Hello'           # Single quotes
s = """Multi
line"""               # Triple quotes
s = r"C:\new\folder"  # Raw string (no escapes)
s = "Hello" "World"   # Concatenation at compile time
```

### Common Operations
```python
s = "Hello, World!"

len(s)              # 13
s[0]                # 'H'
s[-1]               # '!'
s[0:5]              # 'Hello'
"World" in s        # True
s.upper()           # 'HELLO, WORLD!'
s.lower()           # 'hello, world!'
s.split(", ")       # ['Hello', 'World!']
", ".join(['a','b'])  # 'a, b'
s.replace("World", "Python")  # 'Hello, Python!'
s.strip()           # Remove whitespace
```

### String Formatting
```python
name = "Alice"
age = 30

# f-strings (Python 3.6+)
f"Name: {name}, Age: {age}"

# format()
"Name: {}, Age: {}".format(name, age)

# % operator
"Name: %s, Age: %d" % (name, age)
```

---

## Key Concepts

### Immutability
```python
s = "hello"
# s[0] = "H"  # TypeError! Strings are immutable

# Create new string instead
s = "H" + s[1:]  # "Hello"
```

### Unicode
```python
# Python 3 strings are Unicode
s = "Hello, 世界! 🌍"
len(s)  # 12 (counts characters, not bytes)

# Encoding
s.encode('utf-8')  # b'Hello, \xe4\xb8\x96\xe7\x95\x8c! \xf0\x9f\x8c\x8d'

# Decoding
b'Hello'.decode('utf-8')  # 'Hello'
```

### String Interning
```python
# Python interns some strings for efficiency
a = "hello"
b = "hello"
a is b  # True (same object)

# But not all strings
a = "hello world"
b = "hello world"
a is b  # Usually False
```

---

## Next Steps
Start with [String Basics](01_string_basics.md).
