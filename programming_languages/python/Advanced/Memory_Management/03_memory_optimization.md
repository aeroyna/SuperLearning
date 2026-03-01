# Memory Optimization

Python objects have a high overhead. An empty `int` takes 24-28 bytes. Here are techniques to optimize memory usage.

## 1. `__slots__`
Eliminates the `__dict__` dynamic dictionary for instances, saving massive amounts of memory for millions of small objects.

```python
class Point:
    __slots__ = ['x', 'y']
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

## 2. Generators
Process data one item at a time instead of loading entire lists into RAM.

## 3. `interning`
Python automatically interns small integers (-5 to 256) and some strings (identifiers). You can support manual string interning to deduplicate strings.

```python
import sys
a = sys.intern("very_long_string_that_repeats...")
b = sys.intern("very_long_string_that_repeats...")
print(a is b) # True (Shared memory)
```

## 4. `array` module
For lists of uniform numbers, `array.array` is much more compact than `list`.

```python
import array
# 100 signed integers
arr = array.array('i', range(100))
```

## 5. View Slicing (`memoryview`)
Avoid copying bytes when slicing.

```python
data = b'x' * 1000000
mv = memoryview(data)
chunk = mv[0:100] # Zero-copy
```
