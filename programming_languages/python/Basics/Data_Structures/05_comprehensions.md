# Comprehensions

Comprehensions provide a concise, readable syntax for creating collections.

---

## 1. List Comprehension

```python
# Basic syntax: [expression for item in iterable]
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition: [expression for item in iterable if condition]
evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]

# Transform with condition
even_squares = [x**2 for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]
```

### Equivalent Loop
```python
# Comprehension
squares = [x**2 for x in range(10)]

# Equivalent loop
squares = []
for x in range(10):
    squares.append(x**2)
```

---

## 2. Dict Comprehension

```python
# Basic syntax: {key_expr: value_expr for item in iterable}
squares = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# With condition
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}
# {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# From lists
keys = ["a", "b", "c"]
values = [1, 2, 3]
d = {k: v for k, v in zip(keys, values)}
# {'a': 1, 'b': 2, 'c': 3}

# Invert dictionary
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b', 3: 'c'}
```

---

## 3. Set Comprehension

```python
# Basic syntax: {expression for item in iterable}
squares = {x**2 for x in range(-5, 6)}
# {0, 1, 4, 9, 16, 25}  — duplicates removed

# With condition
evens = {x for x in range(10) if x % 2 == 0}
# {0, 2, 4, 6, 8}

# From text
unique_chars = {char.lower() for char in text if char.isalpha()}
```

---

## 4. Generator Expression

```python
# Basic syntax: (expression for item in iterable)
# Returns a generator, not a list!
squares = (x**2 for x in range(10))
# <generator object <genexpr> at 0x...>

# Iterate over it
for s in squares:
    print(s)

# Convert to list
list(squares)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Use directly in functions
sum(x**2 for x in range(10))  # 285
max(x**2 for x in range(10))  # 81
```

### Generator vs List Comprehension
```python
# List: creates entire list in memory
[x**2 for x in range(1000000)]  # ~8MB

# Generator: lazy evaluation, minimal memory
(x**2 for x in range(1000000))  # ~100 bytes

# Use generator when:
# - Processing large data
# - Only iterating once
# - Memory is a concern
```

---

## 5. Nested Comprehensions

### Flatten a Matrix
```python
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

# Flatten: read left to right as nested loops
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Equivalent loop
flat = []
for row in matrix:
    for num in row:
        flat.append(num)
```

### Create a Matrix
```python
# 3x3 matrix of zeros
matrix = [[0 for _ in range(3)] for _ in range(3)]
# [[0, 0, 0], [0, 0, 0], [0, 0, 0]]

# Multiplication table
table = [[i * j for j in range(1, 6)] for i in range(1, 6)]
```

### Nested with Condition
```python
# All pairs where sum is even
pairs = [(x, y) for x in range(5) for y in range(5) if (x + y) % 2 == 0]
```

---

## 6. Conditional Expressions in Comprehensions

### If-Else (Transform)
```python
# [true_expr if condition else false_expr for item in iterable]
signs = ["positive" if x > 0 else "non-positive" for x in [-2, -1, 0, 1, 2]]
# ['non-positive', 'non-positive', 'non-positive', 'positive', 'positive']

# Clamp values
clamped = [max(0, min(x, 100)) for x in [-10, 50, 150]]
# [0, 50, 100]
```

### If Only (Filter)
```python
# [expression for item in iterable if condition]
positives = [x for x in [-2, -1, 0, 1, 2] if x > 0]
# [1, 2]
```

### Combined
```python
# Transform some, filter others
result = [x**2 if x > 0 else 0 for x in numbers if x != 0]
```

---

## 7. Walrus Operator in Comprehensions

```python
# Avoid computing expensive function twice
# Without walrus (calls f(x) twice)
result = [f(x) for x in data if f(x) > 0]

# With walrus (calls f(x) once)
result = [y for x in data if (y := f(x)) > 0]

# Multiple uses
result = [(y, y**2) for x in data if (y := f(x)) is not None]
```

---

## 8. Common Patterns

### Filter and Transform
```python
# Get lengths of words longer than 3 characters
lengths = [len(word) for word in words if len(word) > 3]

# Better with walrus
lengths = [n for word in words if (n := len(word)) > 3]
```

### Dictionary from Object Attributes
```python
users = [User("Alice", 30), User("Bob", 25)]
ages = {user.name: user.age for user in users}
```

### Transpose Matrix
```python
matrix = [[1, 2, 3], [4, 5, 6]]
transposed = [[row[i] for row in matrix] for i in range(len(matrix[0]))]
# [[1, 4], [2, 5], [3, 6]]
```

### Counting with Dict Comprehension
```python
text = "hello world"
counts = {char: text.count(char) for char in set(text)}
```

---

## 9. When to Use Comprehensions

### Do Use Comprehensions When:
```python
# Simple transformations
doubled = [x * 2 for x in numbers]

# Simple filtering
evens = [x for x in numbers if x % 2 == 0]

# Creating dictionaries
d = {k: v for k, v in pairs}
```

### Avoid Comprehensions When:
```python
# Complex logic (use loop instead)
# Bad: hard to read
result = [
    complex_transform(x)
    for x in data
    if condition1(x) and condition2(x)
    if not exception_case(x)
]

# Good: use loop
result = []
for x in data:
    if not (condition1(x) and condition2(x)):
        continue
    if exception_case(x):
        continue
    result.append(complex_transform(x))

# Side effects (use loop)
# Bad: comprehension for side effects
[print(x) for x in items]  # Creates useless list

# Good: use loop
for x in items:
    print(x)
```

---

## 10. Performance Tips

```python
# Generator for large data
sum(x**2 for x in range(1000000))  # Better
sum([x**2 for x in range(1000000)])  # Creates full list

# Local variable for repeated access
# Slower
[data.items[i].value for i in range(len(data.items))]

# Faster
items = data.items
[items[i].value for i in range(len(items))]

# Built-ins are fast
# Instead of: [str(x) for x in numbers]
list(map(str, numbers))  # Often faster
```

---

## 11. Practice Problems

1. Create a list of tuples `(number, square, cube)` for numbers 1-10.

2. Flatten a 3D list (list of lists of lists).

3. Create a dictionary mapping words to their lengths, filtering words under 4 characters.

4. Use a generator expression to find the first number divisible by both 7 and 11 in a range.

---

## Summary

| Type | Syntax | Returns |
|------|--------|---------|
| List | `[expr for x in iter]` | List |
| Dict | `{k: v for x in iter}` | Dict |
| Set | `{expr for x in iter}` | Set |
| Generator | `(expr for x in iter)` | Generator |

Comprehensions are a powerful, Pythonic tool when used appropriately. Keep them simple and readable.
