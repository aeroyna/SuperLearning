# Loop Control Statements

## 1. `break` Statement

Exits the loop immediately:

```python
for i in range(10):
    if i == 5:
        break
    print(i)
# Prints: 0, 1, 2, 3, 4

# Search and exit
for item in items:
    if item == target:
        print(f"Found: {item}")
        break
```

### Breaking Out of Nested Loops
```python
# Problem: break only exits innermost loop
for i in range(3):
    for j in range(3):
        if i == 1 and j == 1:
            break  # Only breaks inner loop
        print(f"({i}, {j})")

# Solution 1: Use a flag
found = False
for i in range(3):
    for j in range(3):
        if i == 1 and j == 1:
            found = True
            break
    if found:
        break

# Solution 2: Move to a function
def search():
    for i in range(3):
        for j in range(3):
            if i == 1 and j == 1:
                return (i, j)
    return None

# Solution 3: Use exception (for complex cases)
class BreakOutOfLoop(Exception):
    pass

try:
    for i in range(3):
        for j in range(3):
            if i == 1 and j == 1:
                raise BreakOutOfLoop()
except BreakOutOfLoop:
    pass
```

---

## 2. `continue` Statement

Skips to the next iteration:

```python
for i in range(10):
    if i % 2 == 0:
        continue  # Skip even numbers
    print(i)
# Prints: 1, 3, 5, 7, 9

# Skip invalid items
for item in items:
    if not is_valid(item):
        continue
    process(item)
```

### With `while`
```python
i = 0
while i < 10:
    i += 1
    if i % 2 == 0:
        continue
    print(i)
# Prints: 1, 3, 5, 7, 9
```

---

## 3. `pass` Statement

Does nothing — a placeholder:

```python
# Empty function (not implemented yet)
def not_implemented():
    pass

# Empty class
class EmptyClass:
    pass

# Empty loop body
for item in items:
    pass  # TODO: implement later

# Empty conditional branch
if condition:
    do_something()
elif other_condition:
    pass  # Explicitly do nothing
else:
    do_other()
```

### `pass` vs `...` (Ellipsis)
```python
# Both work as placeholders
def func1():
    pass

def func2():
    ...

# Ellipsis is often used for type stubs and abstract methods
class AbstractClass:
    def abstract_method(self) -> None:
        ...
```

---

## 4. Loop `else` Clause

The `else` block runs if the loop completes without `break`:

### With `for`
```python
# Search pattern
for item in items:
    if item == target:
        print("Found!")
        break
else:
    print("Not found")  # Runs if break never executed

# Practical example: find a divisor
def find_divisor(n):
    for i in range(2, n):
        if n % i == 0:
            return i  # Found a divisor
    else:
        return None  # No divisor found (n is prime)
```

### With `while`
```python
attempts = 3
while attempts > 0:
    password = input("Enter password: ")
    if password == "secret":
        print("Access granted")
        break
    attempts -= 1
else:
    print("Too many failed attempts")  # Runs if while condition becomes False
```

### Common Use Cases
```python
# 1. Search with not-found handling
for user in users:
    if user.id == target_id:
        found_user = user
        break
else:
    raise ValueError("User not found")

# 2. Validation
for char in password:
    if char.isdigit():
        break
else:
    raise ValueError("Password must contain a digit")

# 3. Prime check
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True  # Could use else here too
```

---

## 5. Combining Control Statements

```python
for i in range(100):
    if i < 10:
        continue  # Skip single digits

    if i > 50:
        break  # Stop after 50

    if i % 7 == 0:
        print(f"{i} is divisible by 7")
else:
    print("Completed without break")  # Won't run (break at 50)
```

---

## 6. Best Practices

### Use Early `continue` for Clarity
```python
# Instead of deeply nested conditions:
for item in items:
    if condition1:
        if condition2:
            if condition3:
                process(item)

# Use continue to flatten:
for item in items:
    if not condition1:
        continue
    if not condition2:
        continue
    if not condition3:
        continue
    process(item)
```

### Prefer Comprehensions When Possible
```python
# Instead of:
result = []
for x in items:
    if x > 0:
        result.append(x * 2)

# Use:
result = [x * 2 for x in items if x > 0]
```

### Avoid Infinite Loops
```python
# Dangerous: no way to exit
while True:
    do_something()

# Safe: has exit condition
while True:
    result = do_something()
    if result.done:
        break

# Safer: explicit condition
max_iterations = 1000
i = 0
while not done and i < max_iterations:
    done = do_something()
    i += 1
```

---

## 7. Practice Problems

1. Print numbers 1-20, skip multiples of 3, stop if number > 15.

2. Find the first number divisible by both 7 and 11 in range 1-200.

3. Implement a retry loop that tries 3 times before giving up.

---

## Next Steps
- [Pattern Matching](04_pattern_matching.md)
