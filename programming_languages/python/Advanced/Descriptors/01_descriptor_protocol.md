# Descriptor Protocol

Descriptors are the mechanism behind properties, methods, static methods, class methods, and `super()`. They let objects customize attribute lookup, storage, and deletion.

## The Protocol

To be a descriptor, a class must implement at least one of these:

*   `__get__(self, obj, objtype=None)`
*   `__set__(self, obj, value)`
*   `__delete__(self, obj)`

## Example: Type-Checked Attribute

```python
class Integer:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):
        if not isinstance(value, int):
            raise TypeError(f"{self.name} must be an integer")
        instance.__dict__[self.name] = value

class Point:
    x = Integer("x")
    y = Integer("y")

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
p.x = 100  # OK
# p.x = "foo" # TypeError: x must be an integer
```

## Data vs Non-Data Descriptors
*   **Data Descriptor**: Defines `__set__` or `__delete__`. Takes precedence over instance dictionary.
*   **Non-Data Descriptor**: Only defines `__get__` (e.g., methods). Instance dictionary takes precedence.
