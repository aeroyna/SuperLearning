# Reference Counting

The primary memory management mechanism in Python (CPython) is **Reference Counting**.

## How it Works
Every object in Python contains a field (ob_refcnt) that counts how many references point to it.

1.  **Increment**: When an object is assigned to a variable, passed as an argument, or added to a list.
2.  **Decrement**: When a variable is deleted, goes out of scope, or is reassigned.

## Immediate Deallocation
When the reference count drops to **zero**, the memory is **immediately** reclaimed. This is deterministic.

```python
import sys

x = []
print(sys.getrefcount(x)) # 2 (x + argument to getrefcount)

y = x
print(sys.getrefcount(x)) # 3

del y
print(sys.getrefcount(x)) # 2
```

## The Limitations
Reference counting is simple and fast, but it cannot handle **Cyclic References**.

```python
a = []
b = []
a.append(b)
b.append(a)

del a
del b
# Ref counts are not zero (they point to each other)
# Memory leak if relying solely on ref counting!
```
This is where the Garbage Collector comes in.
