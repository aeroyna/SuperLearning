# Control Flow

Control flow statements determine the order in which code is executed. Python provides conditionals, loops, and control statements to direct program flow.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Conditional Statements**](01_conditionals.md) | if, elif, else, and ternary operator |
| [**2. Loops**](02_loops.md) | for, while, and iteration patterns |
| [**3. Loop Control**](03_loop_control.md) | break, continue, pass, and else clauses |
| [**4. Pattern Matching**](04_pattern_matching.md) | match/case statements (Python 3.10+) |

---

## Quick Reference

### Conditionals
```python
if condition:
    # code
elif other_condition:
    # code
else:
    # code

# Ternary
result = "yes" if condition else "no"
```

### Loops
```python
# For loop
for item in iterable:
    # code

# While loop
while condition:
    # code

# Range-based
for i in range(10):
    # code
```

### Control Statements
```python
break      # Exit loop immediately
continue   # Skip to next iteration
pass       # Do nothing (placeholder)
```

---

## Truthiness in Conditions

Python evaluates any object in a boolean context:

```python
# Falsy values
if not None:        pass  # None
if not False:       pass  # False
if not 0:           pass  # Zero
if not "":          pass  # Empty string
if not []:          pass  # Empty list
if not {}:          pass  # Empty dict

# Truthy: everything else
if 1:               pass  # Non-zero number
if "hello":         pass  # Non-empty string
if [1, 2]:          pass  # Non-empty list
```

---

## Common Patterns

### Guard Clauses
```python
def process(data):
    if not data:
        return None

    if not valid(data):
        return None

    # Main logic here
    return result
```

### Iteration with Index
```python
for i, item in enumerate(items):
    print(f"{i}: {item}")
```

### Dictionary Iteration
```python
for key, value in my_dict.items():
    print(f"{key}: {value}")
```

---

## Next Steps
Start with [Conditional Statements](01_conditionals.md).
