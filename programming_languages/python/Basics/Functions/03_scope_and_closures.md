# Scope and Closures

## 1. Namespaces

A namespace is a mapping from names to objects. Python has several:

| Namespace | Description | Lifetime |
|-----------|-------------|----------|
| **Built-in** | `print`, `len`, `int`, etc. | Python session |
| **Global** | Module-level names | Module import |
| **Enclosing** | Names in outer functions | Outer function call |
| **Local** | Names in current function | Function call |

```python
# View namespaces
print(dir(__builtins__))  # Built-in names
print(dir())              # Global names
print(locals())           # Local names
print(globals())          # Global names as dict
```

---

## 2. The LEGB Rule

Python looks up names in this order:

1. **L**ocal — Inside the current function
2. **E**nclosing — In enclosing functions
3. **G**lobal — Module level
4. **B**uilt-in — Python's built-in names

```python
x = "global"  # Global

def outer():
    x = "enclosing"  # Enclosing

    def inner():
        x = "local"  # Local
        print(x)     # "local"

    inner()
    print(x)         # "enclosing"

outer()
print(x)             # "global"
```

### Lookup Example
```python
# Built-in
len([1, 2, 3])  # Finds 'len' in built-in namespace

# Shadow built-in (don't do this!)
len = 10  # Now 'len' found in global namespace
# len([1, 2, 3])  # TypeError: 'int' object is not callable

del len  # Remove from global, built-in 'len' accessible again
```

---

## 3. Local Scope

Variables defined inside a function are local:

```python
def func():
    x = 10  # Local to func
    print(x)

func()  # 10
# print(x)  # NameError: name 'x' is not defined
```

### Parameters are Local
```python
def greet(name):  # 'name' is local
    greeting = "Hello"  # Also local
    return f"{greeting}, {name}!"
```

### Local Assignment Creates Local Variable
```python
x = 10  # Global

def func():
    x = 20  # New local 'x', doesn't affect global
    print(x)  # 20

func()
print(x)  # 10 — global unchanged
```

---

## 4. The `global` Keyword

Access and modify global variables from within a function:

```python
count = 0  # Global

def increment():
    global count  # Declare intention to use global
    count += 1

increment()
increment()
print(count)  # 2
```

### Common Use Case: Counters
```python
call_count = 0

def tracked_function():
    global call_count
    call_count += 1
    # ... do work ...

tracked_function()
tracked_function()
print(f"Called {call_count} times")
```

### Why Avoid Global
```python
# Hard to test, hard to reason about
count = 0

def increment():
    global count
    count += 1

# Better: use return values
def increment(count):
    return count + 1

count = 0
count = increment(count)
```

---

## 5. The `nonlocal` Keyword

Modify variables in enclosing (not global) scope:

```python
def outer():
    x = 10

    def inner():
        nonlocal x  # Refer to enclosing x
        x += 1
        print(f"Inner: x = {x}")

    inner()
    print(f"Outer: x = {x}")

outer()
# Inner: x = 11
# Outer: x = 11
```

### Use Case: Closures with State
```python
def make_counter():
    count = 0

    def counter():
        nonlocal count
        count += 1
        return count

    return counter

counter = make_counter()
print(counter())  # 1
print(counter())  # 2
print(counter())  # 3
```

---

## 6. Closures

A closure is a function that remembers values from its enclosing scope:

```python
def make_multiplier(n):
    def multiplier(x):
        return x * n  # 'n' is "closed over"
    return multiplier

double = make_multiplier(2)
triple = make_multiplier(3)

double(5)  # 10
triple(5)  # 15
```

### How Closures Work
```python
def make_multiplier(n):
    def multiplier(x):
        return x * n
    return multiplier

double = make_multiplier(2)

# The closure stores references to enclosed variables
print(double.__closure__)
# (<cell at 0x...: int object at 0x...>,)

print(double.__closure__[0].cell_contents)
# 2
```

### Closure vs Class
```python
# Closure version
def make_accumulator(initial=0):
    total = initial

    def add(value):
        nonlocal total
        total += value
        return total

    return add

acc = make_accumulator()
acc(5)   # 5
acc(10)  # 15

# Class version
class Accumulator:
    def __init__(self, initial=0):
        self.total = initial

    def add(self, value):
        self.total += value
        return self.total

acc = Accumulator()
acc.add(5)   # 5
acc.add(10)  # 15
```

---

## 7. Common Closure Patterns

### Factory Functions
```python
def make_power(exponent):
    def power(base):
        return base ** exponent
    return power

square = make_power(2)
cube = make_power(3)

square(5)  # 25
cube(5)    # 125
```

### Callback with Context
```python
def make_callback(message):
    def callback(result):
        print(f"{message}: {result}")
    return callback

on_success = make_callback("Success")
on_error = make_callback("Error")

on_success("Data loaded")  # "Success: Data loaded"
on_error("Connection failed")  # "Error: Connection failed"
```

### Memoization
```python
def memoize(func):
    cache = {}

    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]

    return wrapper

@memoize
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

fibonacci(100)  # Fast with memoization
```

---

## 8. Closure Gotcha: Late Binding

```python
# Problem: all functions share the same 'i'
funcs = []
for i in range(3):
    funcs.append(lambda: i)

[f() for f in funcs]  # [2, 2, 2] — not [0, 1, 2]!

# Solution 1: default argument
funcs = []
for i in range(3):
    funcs.append(lambda i=i: i)

[f() for f in funcs]  # [0, 1, 2]

# Solution 2: factory function
def make_func(i):
    return lambda: i

funcs = [make_func(i) for i in range(3)]
[f() for f in funcs]  # [0, 1, 2]
```

---

## 9. Free Variables

Variables used in a function but not defined there:

```python
def outer():
    x = 10  # Free variable for inner

    def inner():
        print(x)  # x is a free variable here

    print(inner.__code__.co_freevars)  # ('x',)
    return inner
```

---

## 10. Best Practices

### Minimize Global Variables
```python
# Bad: global state
counter = 0

def increment():
    global counter
    counter += 1

# Good: encapsulated state
def make_counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment
```

### Prefer Parameters Over Closures
```python
# Harder to test (relies on closure)
def make_adder(x):
    def add(y):
        return x + y
    return add

# Easier to test (explicit parameters)
def add(x, y):
    return x + y
```

### Document Closures Clearly
```python
def rate_limiter(max_calls, period):
    """
    Create a rate-limited wrapper function.

    The returned function tracks calls and raises an error
    if called more than max_calls times within period seconds.
    """
    calls = []

    def wrapper(func):
        # ... implementation
        pass

    return wrapper
```

---

## 11. Practice Problems

1. Write a `make_averager()` that returns a function to compute running average.

2. Write a closure that implements a simple cache with max size.

3. Explain what happens here and fix it:
```python
functions = [lambda x: x + i for i in range(5)]
[f(0) for f in functions]
```

---

## Next Steps
- [Lambda Functions](04_lambda_functions.md)
