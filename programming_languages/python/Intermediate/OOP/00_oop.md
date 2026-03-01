# Object-Oriented Programming

OOP in Python is flexible and powerful, supporting classes, inheritance, polymorphism, and encapsulation while maintaining Python's philosophy of simplicity.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Classes and Objects**](01_classes_and_objects.md) | Creating classes, instantiation, attributes |
| [**2. Inheritance**](02_inheritance.md) | Single, multiple, MRO, super() |
| [**3. Encapsulation**](03_encapsulation.md) | Public, protected, private, properties |
| [**4. Polymorphism**](04_polymorphism.md) | Duck typing, abstract classes |
| [**5. Special Methods**](05_special_methods.md) | Magic/dunder methods |
| [**6. Class vs Instance**](06_class_vs_instance.md) | Class attributes, methods, staticmethod, classmethod |

---

## Quick Reference

### Basic Class
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hello, I'm {self.name}"

person = Person("Alice", 30)
print(person.greet())
```

### Inheritance
```python
class Employee(Person):
    def __init__(self, name, age, employee_id):
        super().__init__(name, age)
        self.employee_id = employee_id

    def greet(self):
        return f"{super().greet()}, employee #{self.employee_id}"
```

### Properties
```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius must be positive")
        self._radius = value

    @property
    def area(self):
        return 3.14159 * self._radius ** 2
```

### Special Methods
```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other):
        return self.x == other.x and self.y == other.y
```

---

## OOP Principles in Python

| Principle | Python Implementation |
|-----------|----------------------|
| **Encapsulation** | Naming conventions (`_protected`, `__private`), properties |
| **Inheritance** | Single and multiple inheritance, MRO |
| **Polymorphism** | Duck typing, abstract base classes |
| **Abstraction** | ABC module, abstract methods |

---

## Dataclasses (Python 3.7+)

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float

    def distance_from_origin(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

p = Point(3, 4)
print(p)  # Point(x=3, y=4)
print(p.distance_from_origin())  # 5.0
```

---

## Next Steps
Start with [Classes and Objects](01_classes_and_objects.md).
