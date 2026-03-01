# Cython

Cython is a superset of Python that compiles to C. It gives you C-like performance with Python-like syntax.

## Static Typing
The big speedup comes from adding static type declarations (`cdef`).

### Pure Python
```python
def fib(n):
    a, b = 0, 1
    for i in range(n):
        a, b = a + b, a
    return a
```

### Cython
```python
# my_module.pyx
def fib(int n):
    cdef int i
    cdef double a = 0, b = 1
    for i in range(n):
        a, b = a + b, a
    return a
```

## Compilation
Cython files (`.pyx`) must be compiled.
1.  Create `setup.py`.
2.  Run `python setup.py build_ext --inplace`.
3.  Import the generated `.so` (Linux) or `.pyd` (Windows) file.

## Usage
Use Cython for **CPU-bound** bottlenecks where pure Python is too slow (e.g., number crunching, image processing).
