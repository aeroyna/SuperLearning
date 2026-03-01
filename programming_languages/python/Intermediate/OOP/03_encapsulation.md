# Encapsulation

Encapsulation restricts access to certain components of an object, bundling data and methods that work on that data. Unlike Java or C++, Python does not enforce strict access control (no `private` or `protected` keywords). Instead, it relies on conventions and name mangling.

> *"We are all consenting adults here."* — Python Philosophy

---

## Conventions

### Public
No prefix. Accessible from anywhere.
```python
class User:
    def __init__(self, name):
        self.name = name  # Public
```

### Protected (`_underscore`)
Single leading underscore. Indicates "internal use only" or "non-public". It is a **convention**—interpreted by programmers, not enforced by the interpreter (except in `from module import *`).

```python
class User:
    def __init__(self, name):
        self._name = name  # Warning: Internal use
```

### Private (`__double_underscore`)
Double leading underscore. Python performs **name mangling** to make it harder (but not impossible) to access.

```python
class User:
    def __init__(self, name):
        self.__name = name

u = User("Alice")
# u.__name  # AttributeError!
```

#### Name Mangling Internals
Python rewrites `__name` to `_ClassName__name` to prevent accidental overriding in subclasses.

```python
print(u._User__name)  # "Alice" (Access is still possible)
```

**When to use:** Use `__` primarily to avoid name clashes in inheritance hierarchies, not to secure data.

---

## Properties (`@property`)

The Pythonic way to implement getters and setters. It allows you to start with public attributes and change to getter/setter logic later without breaking the API.

```python
class Temperature:
    def __init__(self, celsius):
        self._celsius = celsius

    @property
    def fahrenheit(self):
        return (self._celsius * 9/5) + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self._celsius = (value - 32) * 5/9

t = Temperature(0)
print(t.fahrenheit)  # 32.0  (Calls getter)
t.fahrenheit = 100   #       (Calls setter)
print(t._celsius)    # 37.77
```

### Benefits
1.  **Validation**: Add checks in setter.
2.  **Computed Attributes**: Calculate values on the fly.
3.  **API Stability**: Change implementation without changing interface (dot notation).

---

## `__slots__`

By default, Python objects store attributes in a dictionary `__dict__`, which consumes significant memory. `__slots__` tells Python to allocate space for a fixed set of attributes, eliminating `__dict__`.

```python
class Point:
    __slots__ = ['x', 'y']

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
# p.z = 3  # AttributeError: 'Point' object has no attribute 'z'
```

### Internals
*   **Memory**: Uses an array of pointers instead of a hash map. Can save ~40-50% memory for millions of small objects.
*   **Performance**: Attribute access is slightly faster.
*   **Side Effect**: Cannot add new attributes at runtime.

---

## Best Practices

1.  **Start Public**: Use simple attributes (`self.x`).
2.  **Refactor to Properties**: If you need validation or computed values later, switch to `@property`.
3.  **Avoid Excessive Getters/Setters**: Don't write Java-style `get_x()` and `set_x()` in Python.
4.  **Use `_` for Internals**: Signal intent to other usages.
