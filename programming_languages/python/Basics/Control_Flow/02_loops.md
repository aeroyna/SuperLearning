# Loops

## 1. The `for` Loop

Python's `for` loop iterates over any **iterable** object:

```python
# List
for item in [1, 2, 3]:
    print(item)

# String
for char in "hello":
    print(char)

# Tuple
for x in (1, 2, 3):
    print(x)

# Dictionary (iterates over keys)
for key in {"a": 1, "b": 2}:
    print(key)

# Set
for item in {1, 2, 3}:
    print(item)

# Range
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4
```

---

## 2. The `range()` Function

Generates a sequence of numbers:

```python
# range(stop) — 0 to stop-1
range(5)         # 0, 1, 2, 3, 4

# range(start, stop) — start to stop-1
range(2, 5)      # 2, 3, 4

# range(start, stop, step)
range(0, 10, 2)  # 0, 2, 4, 6, 8
range(10, 0, -1) # 10, 9, 8, 7, 6, 5, 4, 3, 2, 1

# range is lazy — doesn't create list in memory
r = range(1000000)  # Instant, uses minimal memory
```

### Common Patterns
```python
# Classic index loop
for i in range(len(items)):
    print(f"{i}: {items[i]}")

# Better: use enumerate
for i, item in enumerate(items):
    print(f"{i}: {item}")

# Counting down
for i in range(10, 0, -1):
    print(i)  # 10, 9, 8, ..., 1
```

---

## 3. The `while` Loop

Continues while condition is true:

```python
count = 0
while count < 5:
    print(count)
    count += 1
# Prints: 0, 1, 2, 3, 4

# Infinite loop (use with break)
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input == 'quit':
        break
```

### When to Use `while` vs `for`
```python
# Use for: iterating over a known sequence
for item in items:
    process(item)

# Use while: unknown number of iterations
while not done:
    result = do_work()
    if result.is_complete:
        done = True
```

---

## 4. Loop Iteration Patterns

### `enumerate()` — Index and Value
```python
fruits = ["apple", "banana", "cherry"]

for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
# 0: apple
# 1: banana
# 2: cherry

# Start from different index
for i, fruit in enumerate(fruits, start=1):
    print(f"{i}: {fruit}")
# 1: apple
# 2: banana
# 3: cherry
```

### `zip()` — Parallel Iteration
```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]

for name, age in zip(names, ages):
    print(f"{name} is {age}")

# Stops at shortest iterable
a = [1, 2, 3]
b = [10, 20]
list(zip(a, b))  # [(1, 10), (2, 20)]

# Use zip_longest for padding
from itertools import zip_longest
list(zip_longest(a, b, fillvalue=0))  # [(1, 10), (2, 20), (3, 0)]
```

### `reversed()` — Reverse Iteration
```python
for i in reversed(range(5)):
    print(i)  # 4, 3, 2, 1, 0

for char in reversed("hello"):
    print(char)  # o, l, l, e, h
```

### `sorted()` — Sorted Iteration
```python
numbers = [3, 1, 4, 1, 5, 9]
for n in sorted(numbers):
    print(n)  # 1, 1, 3, 4, 5, 9

# Reverse sort
for n in sorted(numbers, reverse=True):
    print(n)  # 9, 5, 4, 3, 1, 1

# Custom sort key
words = ["apple", "Banana", "cherry"]
for w in sorted(words, key=str.lower):
    print(w)  # apple, Banana, cherry
```

### Dictionary Iteration
```python
d = {"a": 1, "b": 2, "c": 3}

# Keys (default)
for key in d:
    print(key)

# Values
for value in d.values():
    print(value)

# Key-value pairs
for key, value in d.items():
    print(f"{key}: {value}")
```

---

## 5. Nested Loops

```python
# Matrix iteration
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]

for row in matrix:
    for item in row:
        print(item, end=" ")
    print()  # New line after each row

# With indices
for i, row in enumerate(matrix):
    for j, item in enumerate(row):
        print(f"matrix[{i}][{j}] = {item}")
```

### Flattening with itertools
```python
from itertools import chain

matrix = [[1, 2], [3, 4], [5, 6]]
for item in chain.from_iterable(matrix):
    print(item)  # 1, 2, 3, 4, 5, 6
```

---

## 6. Loop Comprehensions

### List Comprehension
```python
# Basic
squares = [x**2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# With condition
evens = [x for x in range(10) if x % 2 == 0]
# [0, 2, 4, 6, 8]

# With transformation and condition
result = [x**2 for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]

# Nested
matrix = [[i*3+j for j in range(3)] for i in range(3)]
# [[0, 1, 2], [3, 4, 5], [6, 7, 8]]

# Flatten
flat = [item for row in matrix for item in row]
# [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

### Dictionary Comprehension
```python
# Basic
squares = {x: x**2 for x in range(5)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Swap keys and values
inverted = {v: k for k, v in original.items()}

# From two lists
keys = ["a", "b", "c"]
values = [1, 2, 3]
d = {k: v for k, v in zip(keys, values)}
```

### Set Comprehension
```python
unique_lengths = {len(word) for word in words}
```

### Generator Expression
```python
# Lazy evaluation — doesn't create list in memory
gen = (x**2 for x in range(1000000))

# Use when you only need to iterate once
sum(x**2 for x in range(1000000))  # Memory efficient
```

---

## 7. The `else` Clause

Both `for` and `while` can have an `else` clause that runs if the loop completes without `break`:

```python
# Search pattern
for item in items:
    if item == target:
        print("Found!")
        break
else:
    print("Not found")

# Prime check
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False  # Not prime
    return True  # No divisors found
```

---

## 8. Performance Considerations

### Prefer Built-ins
```python
# Slow: manual loop
total = 0
for x in numbers:
    total += x

# Fast: built-in sum
total = sum(numbers)
```

### Avoid Repeated Method Lookups
```python
# Slow: method lookup each iteration
for item in items:
    result.append(item.upper())

# Faster: local reference
append = result.append
upper = str.upper
for item in items:
    append(upper(item))

# Best: list comprehension
result = [item.upper() for item in items]
```

### Use `itertools` for Complex Iteration
```python
from itertools import count, cycle, repeat, islice

# Infinite counter
for i in islice(count(start=10, step=2), 5):
    print(i)  # 10, 12, 14, 16, 18

# Cycle through items
colors = cycle(['red', 'green', 'blue'])
for _, color in zip(range(7), colors):
    print(color)

# Repeat value
for x in repeat('hello', 3):
    print(x)
```

---

## 9. Practice Problems

1. Print a multiplication table (1-10).

2. Find all prime numbers up to n using the Sieve of Eratosthenes.

3. Iterate over two lists simultaneously and print pairs.

4. Flatten a 2D list using a comprehension.

---

## Next Steps
- [Loop Control](03_loop_control.md)
