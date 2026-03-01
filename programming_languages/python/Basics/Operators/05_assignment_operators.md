# Assignment Operators

## 1. Basic Assignment

```python
x = 10        # Assign 10 to x
name = "Alice"  # Assign string to name

# Multiple assignment
a = b = c = 0   # All get value 0

# Parallel assignment (tuple unpacking)
x, y = 1, 2     # x=1, y=2
x, y = y, x     # Swap values
```

---

## 2. Augmented Assignment Operators

Combine an operation with assignment:

| Operator | Example | Equivalent To |
|----------|---------|---------------|
| `+=` | `x += 5` | `x = x + 5` |
| `-=` | `x -= 5` | `x = x - 5` |
| `*=` | `x *= 5` | `x = x * 5` |
| `/=` | `x /= 5` | `x = x / 5` |
| `//=` | `x //= 5` | `x = x // 5` |
| `%=` | `x %= 5` | `x = x % 5` |
| `**=` | `x **= 5` | `x = x ** 5` |
| `&=` | `x &= 5` | `x = x & 5` |
| `\|=` | `x \|= 5` | `x = x \| 5` |
| `^=` | `x ^= 5` | `x = x ^ 5` |
| `>>=` | `x >>= 5` | `x = x >> 5` |
| `<<=` | `x <<= 5` | `x = x << 5` |

---

## 3. Augmented Assignment Is NOT Just Syntactic Sugar

For mutable objects, augmented assignment can behave differently:

```python
# With immutable (int)
x = 5
print(id(x))  # 140...
x += 1
print(id(x))  # Different! New object created

# With mutable (list)
lst = [1, 2, 3]
print(id(lst))  # 140...
lst += [4, 5]
print(id(lst))  # Same! Modified in place
```

### The Difference Explained

```python
# For lists:
lst = [1, 2, 3]

# lst += [4] calls lst.__iadd__([4])
# which modifies lst in place

lst2 = lst
lst += [4]
print(lst2)  # [1, 2, 3, 4] — same object!

# lst = lst + [4] creates new list
lst3 = lst
lst = lst + [5]
print(lst3)  # [1, 2, 3, 4] — different objects now
```

### Tuples: Edge Case
```python
t = (1, 2)

# Tuples are immutable, so += creates new tuple
t2 = t
t += (3,)
print(t2)  # (1, 2) — different object
```

---

## 4. Unpacking Assignment

### Basic Unpacking
```python
# Tuple unpacking
x, y, z = (1, 2, 3)

# List unpacking
a, b, c = [1, 2, 3]

# String unpacking
first, second, third = "abc"

# Function return unpacking
def get_point():
    return 10, 20

x, y = get_point()
```

### Extended Unpacking (Python 3)
```python
# Catch remaining elements
first, *rest = [1, 2, 3, 4, 5]
# first = 1, rest = [2, 3, 4, 5]

*start, last = [1, 2, 3, 4, 5]
# start = [1, 2, 3, 4], last = 5

first, *middle, last = [1, 2, 3, 4, 5]
# first = 1, middle = [2, 3, 4], last = 5

# Works with any iterable
a, *b, c = range(10)
# a = 0, b = [1, 2, 3, 4, 5, 6, 7, 8], c = 9
```

### Nested Unpacking
```python
# Unpack nested structures
(a, b), c = [1, 2], 3
# a = 1, b = 2, c = 3

data = [(1, 2), (3, 4), (5, 6)]
for (x, y) in data:
    print(f"{x}, {y}")
```

### Ignoring Values
```python
# Use _ for values you don't need
x, _, z = (1, 2, 3)  # Ignore middle value

# Ignore multiple
first, *_, last = [1, 2, 3, 4, 5]
```

---

## 5. Walrus Operator `:=` (Python 3.8+)

Assigns and returns the value in a single expression.

### Basic Usage
```python
# Without walrus
n = len(data)
if n > 10:
    print(f"Too long: {n}")

# With walrus
if (n := len(data)) > 10:
    print(f"Too long: {n}")
```

### In While Loops
```python
# Without walrus
line = file.readline()
while line:
    process(line)
    line = file.readline()

# With walrus
while (line := file.readline()):
    process(line)
```

### In List Comprehensions
```python
# Filter and use result
results = [y for x in data if (y := expensive_function(x)) is not None]

# Avoid repeated expensive calls
# Without walrus (calls function twice):
results = [f(x) for x in data if f(x) > 0]

# With walrus (calls function once):
results = [y for x in data if (y := f(x)) > 0]
```

### Common Patterns
```python
# Regex matching
import re
if (match := re.search(pattern, text)):
    print(match.group())

# Reading until empty
while (chunk := file.read(8192)):
    process(chunk)

# Finding first match
if (user := find_user(user_id)) is not None:
    send_email(user)
```

### Limitations
```python
# Cannot use at statement level
# x := 5  # SyntaxError!

# Must use parentheses in many contexts
# if x := 5:  # SyntaxError in some cases
if (x := 5):  # OK
    pass
```

---

## 6. Global and Nonlocal Assignment

### Global
```python
count = 0

def increment():
    global count  # Required to modify global
    count += 1

increment()
print(count)  # 1
```

### Nonlocal
```python
def outer():
    x = 0

    def inner():
        nonlocal x  # Required to modify enclosing scope
        x += 1

    inner()
    print(x)  # 1

outer()
```

---

## 7. Best Practices

### Do's
```python
# Use meaningful variable names
user_count = 42

# Use unpacking for multiple values
x, y = get_coordinates()

# Use augmented assignment for readability
total += price

# Use walrus for conditional assignment
if (result := compute()) is not None:
    use(result)
```

### Don'ts
```python
# Avoid chained assignment with mutable objects
# a = b = []  # Both point to SAME list!
a, b = [], []  # Separate lists

# Avoid overusing walrus (can hurt readability)
# result = [(y, z) for x in data if (y := f(x)) and (z := g(y))]

# Avoid global when possible
# Use return values and parameters instead
```

---

## 8. Practice Problems

1. What's the output?
```python
a = b = [1, 2]
a += [3]
print(b)

c = d = 10
c += 1
print(d)
```

2. Use the walrus operator to simplify:
```python
match = re.search(r'\d+', text)
if match:
    number = match.group()
    print(number)
```

3. Unpack this nested structure:
```python
data = ((1, 2), (3, 4, 5), (6,))
# Extract: first_pair=(1,2), middle_elements=[3,4,5], last=6
```

---

## Next Steps
- [Special Operators](06_special_operators.md)
