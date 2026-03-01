# Comprehensions

Comprehensions provide a concise way to create collections based on existing iterables. They are considered "Pythonic" alternatives to `map` and `filter`.

## List Comprehensions
`[expression for item in iterable if condition]`

```python
numbers = [1, 2, 3, 4, 5]
squared_evens = [x**2 for x in numbers if x % 2 == 0]
# Equivalent to map(lambda x: x**2, filter(lambda x: x%2==0, numbers))
print(squared_evens)  # [4, 16]
```

### Nested Comprehensions
You can have multiple `for` clauses (equivalent to nested loops).

```python
matrix = [[1, 2], [3, 4]]
flattened = [num for row in matrix for num in row]
print(flattened)  # [1, 2, 3, 4]
```

## Dictionary Comprehensions
`{key_expr: value_expr for item in iterable}`

```python
names = ["Alice", "Bob"]
name_lengths = {name: len(name) for name in names}
print(name_lengths)  # {'Alice': 5, 'Bob': 3}
```

## Set Comprehensions
`{expression for item in iterable}`

```python
nums = [1, 2, 2, 3, 3, 3]
unique_squares = {x**2 for x in nums}
print(unique_squares)  # {1, 4, 9}
```

## Generator Expressions
`(`expression for item in iterable`)`

Doesn't create a tuple! Creates a **generator object** (lazy evaluation). Memory efficient for large datasets.

```python
# Sum of squares of first 1,000,000 integers
# Doesn't build a list in memory!
total = sum(x**2 for x in range(1000000))
```

## Best Practices
1.  **Keep it simple**: If logic is complex, use a regular `for` loop. Comprehensions should be readable.
2.  **Avoid side effects**: Don't use comprehensions just to call a function (e.g., `[print(x) for x in items]`). Use a loop.
