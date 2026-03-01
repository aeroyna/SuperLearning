# Descriptors

Descriptors are objects that define how attribute access works. They power properties, methods, and more.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Descriptor Protocol**](01_descriptor_protocol.md) | __get__, __set__, __delete__ |
| [**2. Common Descriptors**](02_common_descriptors.md) | property, classmethod, staticmethod |

---

## Quick Reference

### The Descriptor Protocol
```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        print(f"Getting from {obj}")
        return self.value

    def __set__(self, obj, value):
        print(f"Setting {value} on {obj}")
        self.value = value

    def __delete__(self, obj):
        print(f"Deleting from {obj}")
        del self.value

class MyClass:
    attr = Descriptor()

obj = MyClass()
obj.attr = 10  # Calls __set__
print(obj.attr)  # Calls __get__
```

### Data vs Non-Data Descriptors
```python
# Data descriptor: has __set__ or __delete__
# Non-data descriptor: only has __get__

# Lookup order:
# 1. Data descriptors
# 2. Instance __dict__
# 3. Non-data descriptors
# 4. Class __dict__
# 5. __getattr__
```

### Typed Property Example
```python
class Typed:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type}")
        obj.__dict__[self.name] = value

class Person:
    name = Typed('name', str)
    age = Typed('age', int)

    def __init__(self, name, age):
        self.name = name
        self.age = age

p = Person("Alice", 30)
p.age = "thirty"  # TypeError
```

---

## Built-in Descriptors

### property
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        self._radius = value

    @radius.deleter
    def radius(self):
        del self._radius
```

### classmethod and staticmethod
```python
class MyClass:
    @classmethod
    def class_method(cls):
        return cls

    @staticmethod
    def static_method():
        return "static"
```

---

## Next Steps
Start with [Descriptor Protocol](01_descriptor_protocol.md).
