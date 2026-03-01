# Conditional Statements

## 1. The `if` Statement

```python
x = 10

if x > 0:
    print("Positive")
```

### `if-else`
```python
x = -5

if x > 0:
    print("Positive")
else:
    print("Non-positive")
```

### `if-elif-else`
```python
x = 0

if x > 0:
    print("Positive")
elif x < 0:
    print("Negative")
else:
    print("Zero")
```

### Multiple `elif`
```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"
```

---

## 2. Ternary Conditional Expression

```python
# syntax: value_if_true if condition else value_if_false

x = 10
result = "positive" if x > 0 else "non-positive"

# Use in expressions
print(f"x is {'even' if x % 2 == 0 else 'odd'}")

# Can be nested (but avoid if complex)
result = "positive" if x > 0 else "zero" if x == 0 else "negative"
```

### When to Use Ternary
```python
# Good: simple value selection
status = "active" if is_active else "inactive"

# Avoid: complex logic
# Bad:
result = (do_thing_a() if cond_a else do_thing_b()) if main_cond else (do_thing_c() if cond_c else do_thing_d())

# Better: use regular if-elif-else
```

---

## 3. Nested Conditionals

```python
x, y = 5, 10

if x > 0:
    if y > 0:
        print("Both positive")
    else:
        print("x positive, y non-positive")
else:
    if y > 0:
        print("x non-positive, y positive")
    else:
        print("Both non-positive")
```

### Flattening Nested Conditionals
```python
# Nested (harder to read)
if condition1:
    if condition2:
        do_something()

# Flattened (easier to read)
if condition1 and condition2:
    do_something()
```

---

## 4. Compound Conditions

### `and`, `or`, `not`
```python
x, y = 5, 10

# and — both must be true
if x > 0 and y > 0:
    print("Both positive")

# or — at least one must be true
if x > 0 or y > 0:
    print("At least one positive")

# not — inverts the condition
if not (x < 0):
    print("x is not negative")
```

### Chained Comparisons
```python
# Instead of: x > 0 and x < 10
if 0 < x < 10:
    print("x is between 0 and 10")

# Multiple chains
if 0 < x < y < 100:
    print("Valid range")
```

---

## 5. Membership and Identity Tests

### `in` / `not in`
```python
fruits = ["apple", "banana", "cherry"]

if "apple" in fruits:
    print("Found apple")

if "grape" not in fruits:
    print("No grape")

# Works with strings
if "a" in "apple":
    print("Contains 'a'")

# Works with dicts (checks keys)
user = {"name": "Alice", "age": 30}
if "name" in user:
    print(user["name"])
```

### `is` / `is not`
```python
x = None

# Always use 'is' for None
if x is None:
    print("x is None")

if x is not None:
    print("x has a value")
```

---

## 6. Common Patterns

### Guard Clauses (Early Returns)
```python
def process_user(user):
    if user is None:
        return None

    if not user.is_active:
        return None

    if user.is_banned:
        return None

    # Main logic - user is valid
    return user.process()
```

### Default Values
```python
# With or
name = user_input or "Anonymous"

# With ternary
name = user_input if user_input else "Anonymous"

# With if-else (most explicit)
if user_input:
    name = user_input
else:
    name = "Anonymous"
```

### Dictionary-based Dispatch
```python
def handle_add():
    pass
def handle_sub():
    pass
def handle_mul():
    pass

operations = {
    '+': handle_add,
    '-': handle_sub,
    '*': handle_mul,
}

op = '+'
if op in operations:
    operations[op]()
else:
    raise ValueError(f"Unknown operation: {op}")
```

### Walrus Operator in Conditions
```python
# Without walrus
match = re.search(pattern, text)
if match:
    print(match.group())

# With walrus (Python 3.8+)
if (match := re.search(pattern, text)):
    print(match.group())

# In comprehensions
valid = [y for x in data if (y := process(x)) is not None]
```

---

## 7. Truthiness Edge Cases

### Empty vs None
```python
def process(items):
    # Distinguish between None and empty list
    if items is None:
        print("No list provided")
    elif not items:
        print("Empty list")
    else:
        print(f"Got {len(items)} items")

process(None)   # "No list provided"
process([])     # "Empty list"
process([1,2])  # "Got 2 items"
```

### Zero is Falsy
```python
count = 0

# Bug: treats 0 as "no count"
if count:
    print(f"Count: {count}")

# Fix: explicit check
if count is not None:
    print(f"Count: {count}")
```

---

## 8. Best Practices

### Do
```python
# Use positive conditions when possible
if is_valid:
    process()

# Use 'is' for None
if value is None:
    pass

# Use chained comparisons
if 0 <= index < len(items):
    pass
```

### Don't
```python
# Avoid comparing to True/False
if is_valid == True:  # Bad
if is_valid:          # Good

# Avoid deep nesting
if a:
    if b:
        if c:
            do_thing()

# Better: combine or use guard clauses
if a and b and c:
    do_thing()
```

---

## 9. Practice Problems

1. Write a function that returns "child" (0-12), "teen" (13-19), "adult" (20-64), or "senior" (65+) based on age.

2. Write a function that checks if a year is a leap year.

3. Write a function that returns the grade (A/B/C/D/F) given a score, handling invalid inputs.

---

## Next Steps
- [Loops](02_loops.md)
