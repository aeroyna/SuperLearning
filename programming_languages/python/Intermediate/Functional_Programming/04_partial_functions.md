# Partial Functions

Partial function application allows you to "freeze" some portion of a function's arguments and return a new function with fewer arguments.

## `functools.partial`

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Create a new function that always squares its argument
square = partial(power, exponent=2)
# Equivalent to: def square(base): return power(base, exponent=2)

# Create a new function that always cubes
cube = partial(power, exponent=3)

print(square(4))  # 16
print(cube(4))    # 64
```

## Use Case: Simplification
Useful when using APIs that expect functions with a specific signature (e.g., callbacks, or `map`).

```python
def multiply(x, y):
    return x * y

# map only accepts one iterable argument for the function
# We want to multiply everything by 2
doubler = partial(multiply, 2)
result = map(doubler, [1, 2, 3])
print(list(result)) # [2, 4, 6]
```

## Internals
`partial` objects are not traditional functions, but callable objects.
They hold:
*   `func`: The original function
*   `args`: Frozen positional arguments
*   `keywords`: Frozen keyword arguments

They are slightly slower than native closures/lambdas but provide introspection benefits (you can inspect `partial_obj.args`).
