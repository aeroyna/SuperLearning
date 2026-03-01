# First Class Functions

In Python, functions are "first-class citizens". This means they can be treated just like any other variable (integer, string, object).

## Key Capabilities

1.  **Assigned to variables**
2.  **Passed as arguments to other functions** (Higher-Order Functions)
3.  **Returned from other functions**
4.  **Stored in data structures**

### 1. Assigning to Variables
```python
def shout(text):
    return text.upper()

speak = shout  # Assign function object to 'speak'
print(speak("hello"))  # "HELLO"
```
Note: `speak` points to the *same function object* as `shout`.

### 2. Passing as Arguments
This is the basis of callbacks and many functional patterns.

```python
def apply_operation(func, x, y):
    return func(x, y)

def add(a, b): return a + b
def sub(a, b): return a - b

print(apply_operation(add, 5, 3))  # 8
print(apply_operation(sub, 5, 3))  # 2
```

### 3. Returning Functions (Closures)
Functions can define and return other functions.

```python
def create_multiplier(factor):
    def multiplier(x):
        return x * factor
    return multiplier

double = create_multiplier(2)
triple = create_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

### 4. Storing in Structures
```python
operations = {
    "add": add,
    "sub": sub
}

op = operations["add"]
print(op(10, 20))  # 30
```

## Internals
Functions are objects of type `function`. They have attributes like:
*   `__name__`: Name of the function.
*   `__code__`: The compiled bytecode.
*   `__closure__`: Tuple of cells that contain bindings for free variables (if it's a closure).

```python
print(shout.__class__)  # <class 'function'>
```
