# Map, Filter, and Reduce

These are the three pillars of functional programming for processing collections. While list comprehensions are often more "Pythonic", understanding these is essential.

## Map
Applies a function to every item in an iterable. Returns an iterator (lazy).

```python
# map(function, iterable)

numbers = [1, 2, 3, 4]
squared = map(lambda x: x**2, numbers)

print(list(squared))  # [1, 4, 9, 16]
```

## Filter
Keeps items from an iterable where the function returns `True`.

```python
# filter(function, iterable)

numbers = [1, 2, 3, 4, 5, 6]
evens = filter(lambda x: x % 2 == 0, numbers)

print(list(evens))  # [2, 4, 6]
```

## Reduce
Reduces an iterable to a single value by accumulating the result. Moved to `functools` in Python 3.

```python
from functools import reduce

# reduce(function, iterable[, initializer])

numbers = [1, 2, 3, 4]

# Step-by-step:
# ((1 + 2) + 3) + 4 = 10
total = reduce(lambda x, y: x + y, numbers)
print(total)  # 10
```

### With Initializer
Always provide an initializer to handle empty lists cleanly and set the starting type.

```python
# Sum starting at 10
print(reduce(lambda x, y: x + y, [1, 2], 10))  # 13
```

## Performance Note
`map` and `filter` are written in C and can be faster than explicit for-loops. However, list comprehensions are often comparable in speed and more readable in Python.
