# Logical Operators

## 1. The Three Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `and` | True if both operands are true | `True and False` → `False` |
| `or` | True if at least one operand is true | `True or False` → `True` |
| `not` | Inverts the boolean value | `not True` → `False` |

---

## 2. Truth Tables

### `and`
| A | B | A and B |
|---|---|---------|
| True | True | True |
| True | False | False |
| False | True | False |
| False | False | False |

### `or`
| A | B | A or B |
|---|---|--------|
| True | True | True |
| True | False | True |
| False | True | True |
| False | False | False |

### `not`
| A | not A |
|---|-------|
| True | False |
| False | True |

---

## 3. Short-Circuit Evaluation

Python evaluates logical expressions lazily, stopping as soon as the result is determined.

### `and` Short-Circuit
```python
# Stops at first falsy value
False and print("never runs")  # False, print not called

# Real-world use: safe navigation
user and user.profile and user.profile.name

# Check before access
data and len(data) > 0  # Safe even if data is None
```

### `or` Short-Circuit
```python
# Stops at first truthy value
True or print("never runs")  # True, print not called

# Real-world use: default values
name = user_input or "Anonymous"

# Chain of fallbacks
value = primary or secondary or default
```

---

## 4. Return Values (Not Just True/False!)

Python's `and` and `or` return **actual values**, not just booleans.

### `and` Returns
- First falsy value, OR
- Last value if all truthy

```python
1 and 2 and 3      # 3 (last truthy)
1 and 0 and 3      # 0 (first falsy)
"" and "hello"     # "" (first falsy)
"hello" and "world"  # "world" (last truthy)
```

### `or` Returns
- First truthy value, OR
- Last value if all falsy

```python
1 or 2 or 3        # 1 (first truthy)
0 or "" or []      # [] (last falsy)
None or 0 or "hi"  # "hi" (first truthy)
```

### Practical Patterns
```python
# Default value
name = user_name or "Guest"

# Safe access (use with caution)
result = data and data.get('key')

# Conditional execution
debug and print("Debug:", value)  # Only prints if debug is truthy
```

---

## 5. `not` Operator

```python
not True   # False
not False  # True
not 0      # True
not 1      # False
not ""     # True
not "hi"   # False
not []     # True
not [1]    # False
not None   # True
```

### Double Negation
```python
not not True   # True
not not []     # False

# Equivalent to bool()
bool([])       # False
not not []     # False
```

---

## 6. Operator Precedence

From highest to lowest:
1. `not`
2. `and`
3. `or`

```python
# Without parentheses
True or True and False   # True (and binds tighter)
# Equivalent to: True or (True and False)

not True or False       # False
# Equivalent to: (not True) or False

# Use parentheses for clarity!
(True or True) and False  # False
not (True or False)       # False
```

---

## 7. Combining with Comparisons

```python
x = 5

# Range check
if 0 < x < 10:
    print("In range")

# Multiple conditions
if x > 0 and x < 10 and x != 5:
    print("Valid")

# Alternative conditions
if x < 0 or x > 10:
    print("Out of range")

# Complex conditions
if (x > 0 and x < 5) or (x > 10 and x < 15):
    print("In one of the ranges")
```

---

## 8. Common Patterns and Idioms

### Default Value
```python
# Simple default
name = user_input or "default"

# Be careful with falsy valid values!
count = user_count or 10  # Bug if user_count is 0!

# Better: explicit None check
count = user_count if user_count is not None else 10
```

### Guard Clauses
```python
def process(data):
    if not data:
        return None
    # Process data...
```

### All/Any Conditions
```python
# Check if all conditions are true
if cond1 and cond2 and cond3:
    pass

# Using all() for many conditions
conditions = [cond1, cond2, cond3]
if all(conditions):
    pass

# Check if any condition is true
if cond1 or cond2 or cond3:
    pass

# Using any() for many conditions
if any(conditions):
    pass
```

### Ternary Operator
```python
# Simple ternary
result = "yes" if condition else "no"

# Nested (avoid if complex)
result = "a" if x > 0 else "b" if x < 0 else "zero"
```

---

## 9. Boolean Algebra Laws

### De Morgan's Laws
```python
# not (A and B) == (not A) or (not B)
not (True and False) == (not True) or (not False)  # True

# not (A or B) == (not A) and (not B)
not (True or False) == (not True) and (not False)  # True
```

### Useful Equivalences
```python
# Double negation
not not x == bool(x)

# Distribution
A and (B or C) == (A and B) or (A and C)
A or (B and C) == (A or B) and (A or C)

# Identity
A and True == A
A or False == A

# Annihilation
A and False == False
A or True == True

# Complement
A and (not A) == False
A or (not A) == True
```

---

## 10. Practice Problems

1. What does each return?
```python
print(0 or 1 and 2)
print(1 and 2 or 3)
print(not 0 and not "")
```

2. Simplify:
```python
not (x > 5 and x < 10)
```

3. Write a function:
```python
def validate(name, age, email):
    """Return True if name is non-empty, age is 18-120, and email contains @"""
    pass
```

---

## Next Steps
- [Bitwise Operators](04_bitwise_operators.md)
