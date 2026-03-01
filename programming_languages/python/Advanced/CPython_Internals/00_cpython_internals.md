# CPython Internals

Understanding CPython's implementation helps write better Python and debug mysterious behavior.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Python Object Model**](01_object_model.md) | PyObject, types |
| [**2. Bytecode**](02_bytecode.md) | Compilation and execution |
| [**3. The Interpreter**](03_interpreter.md) | Execution model |

---

## Quick Reference

### Bytecode
```python
import dis

def greet(name):
    return f"Hello, {name}!"

dis.dis(greet)
#  2           0 LOAD_CONST               1 ('Hello, ')
#              2 LOAD_FAST                0 (name)
#              4 FORMAT_VALUE             0
#              6 LOAD_CONST               2 ('!')
#              8 BUILD_STRING             3
#             10 RETURN_VALUE
```

### Code Objects
```python
def add(a, b):
    return a + b

code = add.__code__

code.co_varnames   # ('a', 'b')
code.co_consts     # (None,)
code.co_code       # bytecode bytes
code.co_filename   # source file
code.co_firstlineno  # first line number
```

### Object Internals
```python
# Every Python object has:
# - Reference count
# - Type pointer

import sys

x = [1, 2, 3]
sys.getrefcount(x)  # Reference count
type(x)             # Type
id(x)               # Memory address
```

---

## Key Concepts

### Object Header (PyObject)
```c
// Every Python object starts with:
struct PyObject {
    Py_ssize_t ob_refcnt;   // Reference count
    PyTypeObject *ob_type;  // Type pointer
};
```

### Small Integer Cache
```python
a = 256
b = 256
a is b  # True — cached

a = 257
b = 257
a is b  # False — not cached
```

### String Interning
```python
a = "hello"
b = "hello"
a is b  # True — interned

a = "hello world"
b = "hello world"
a is b  # Often False — not interned

import sys
a = sys.intern("hello world")
b = sys.intern("hello world")
a is b  # True — forced interning
```

---

## The GIL

The Global Interpreter Lock:
- Protects Python object state
- Only one thread executes Python bytecode at a time
- Released during I/O operations
- Bypassed by multiprocessing

---

## Next Steps
Start with [Python Object Model](01_object_model.md).
