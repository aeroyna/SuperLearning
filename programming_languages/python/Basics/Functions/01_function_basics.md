# Function Basics

## 1. Defining Functions

```python
def function_name(parameters):
    """Docstring describing the function."""
    # Function body
    return value
```

### Simple Function
```python
def greet():
    print("Hello!")

greet()  # Prints: Hello!
```

### Function with Parameters
```python
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")  # Prints: Hello, Alice!
```

### Function with Return Value
```python
def add(a, b):
    return a + b

result = add(3, 5)  # result = 8
```

---

## 2. Return Values

### Returning a Value
```python
def square(x):
    return x ** 2

result = square(5)  # 25
```

### Returning Multiple Values
```python
def divide(a, b):
    quotient = a // b
    remainder = a % b
    return quotient, remainder  # Returns a tuple

q, r = divide(17, 5)  # q=3, r=2

# Or unpack as tuple
result = divide(17, 5)
print(result)  # (3, 2)
```

### No Return Statement
```python
def greet(name):
    print(f"Hello, {name}!")
    # No return statement

result = greet("Alice")
print(result)  # None
```

### Early Return
```python
def get_grade(score):
    if score < 0 or score > 100:
        return None  # Early return for invalid input

    if score >= 90:
        return 'A'
    elif score >= 80:
        return 'B'
    elif score >= 70:
        return 'C'
    elif score >= 60:
        return 'D'
    else:
        return 'F'
```

---

## 3. Docstrings

### Single-Line Docstring
```python
def add(a, b):
    """Return the sum of a and b."""
    return a + b
```

### Multi-Line Docstring (Google Style)
```python
def calculate_bmi(weight, height):
    """
    Calculate Body Mass Index.

    Args:
        weight: Weight in kilograms.
        height: Height in meters.

    Returns:
        The BMI as a float.

    Raises:
        ValueError: If weight or height is not positive.

    Example:
        >>> calculate_bmi(70, 1.75)
        22.86
    """
    if weight <= 0 or height <= 0:
        raise ValueError("Weight and height must be positive")
    return weight / (height ** 2)
```

### Accessing Docstrings
```python
print(calculate_bmi.__doc__)
help(calculate_bmi)
```

---

## 4. Function Annotations (Type Hints)

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

def add(a: int, b: int) -> int:
    return a + b

def divide(a: float, b: float) -> tuple[float, float]:
    return a // b, a % b

# Annotations are not enforced at runtime
greet(42)  # Works, but type checker would warn
```

### Complex Type Hints
```python
from typing import Optional, List, Dict, Callable

def process(
    items: List[int],
    callback: Callable[[int], int],
    config: Optional[Dict[str, str]] = None
) -> List[int]:
    if config is None:
        config = {}
    return [callback(item) for item in items]
```

---

## 5. Functions as Objects

### Assign to Variable
```python
def greet(name):
    return f"Hello, {name}!"

say_hi = greet  # Assign function to new name
say_hi("World")  # "Hello, World!"
```

### Pass as Argument
```python
def apply_twice(func, value):
    return func(func(value))

def double(x):
    return x * 2

apply_twice(double, 5)  # 20
```

### Return from Function
```python
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)

double(5)  # 10
triple(5)  # 15
```

### Store in Data Structures
```python
def add(a, b):
    return a + b

def sub(a, b):
    return a - b

def mul(a, b):
    return a * b

operations = {
    '+': add,
    '-': sub,
    '*': mul,
}

operations['+'](3, 5)  # 8
```

---

## 6. Nested Functions

```python
def outer():
    def inner():
        print("Inner function")
    inner()  # Call inner function
    print("Outer function")

outer()
# Prints:
# Inner function
# Outer function

# inner()  # Error! inner is not accessible outside outer
```

### Helper Functions
```python
def process_data(data):
    def validate(item):
        return item is not None and item > 0

    def transform(item):
        return item * 2

    return [transform(item) for item in data if validate(item)]

process_data([1, -2, None, 3, 0, 4])  # [2, 6, 8]
```

---

## 7. Function Attributes

```python
def greet(name):
    """A greeting function."""
    return f"Hello, {name}!"

# Built-in attributes
greet.__name__       # 'greet'
greet.__doc__        # 'A greeting function.'
greet.__annotations__  # {} (empty if no type hints)
greet.__module__     # '__main__'

# Custom attributes
greet.version = "1.0"
print(greet.version)  # "1.0"
```

---

## 8. Best Practices

### Single Responsibility
```python
# Bad: does too many things
def process_user(user):
    validate(user)
    save_to_db(user)
    send_email(user)
    log_action(user)

# Good: each function does one thing
def validate_user(user):
    ...

def save_user(user):
    ...

def notify_user(user):
    ...
```

### Descriptive Names
```python
# Bad
def p(x):
    return x * x

# Good
def calculate_square(number):
    return number ** 2
```

### Keep Functions Short
```python
# If a function is too long, split it
def process_order(order):
    validated_order = validate_order(order)
    calculated_order = calculate_totals(validated_order)
    return save_order(calculated_order)
```

### Pure Functions (When Possible)
```python
# Pure: no side effects, same input = same output
def add(a, b):
    return a + b

# Impure: has side effects
total = 0
def add_to_total(x):
    global total
    total += x  # Modifies global state
```

---

## 9. Practice Problems

1. Write a function `is_palindrome(s)` that returns True if a string is a palindrome.

2. Write a function `factorial(n)` that returns n! using recursion.

3. Write a function `fibonacci(n)` that returns the nth Fibonacci number.

4. Write a function that takes a list of numbers and returns a tuple of (min, max, average).

---

## Next Steps
- [Arguments and Parameters](02_arguments.md)
