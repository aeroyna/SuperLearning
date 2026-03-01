# General Optimization

## Algorithms First
An O(n^2) algorithm will always lose to O(n log n) eventually. No amount of micro-optimization fixes bad algorithmic complexity.

## Global Variables
Accessing global variables is slower than local variables (opcode `LOAD_GLOBAL` vs `LOAD_FAST`).
**Tip**: If you loop a lot, assign global functions to local vars.

```python
# Slower
for i in range(1000):
    math.sqrt(i)

# Faster
sqrt = math.sqrt
for i in range(1000):
    sqrt(i)
```

## Dot Lookup overhead
Chained lookups (`a.b.c.d`) trigger multiple dictionary lookups.
Cache deep attributes if accessed in a loop.

## Built-ins
Functions written in C (`sum`, `map`) are usually faster than Python loops.
