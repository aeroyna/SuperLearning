# Polymorphism

Polymorphism allows objects of different classes to be treated as objects of a common superclass. In Python, this is primarily achieved through **Duck Typing**, though explicit interfaces are possible.

## Duck Typing

> *"If it walks like a duck and quacks like a duck, then it must be a duck."*

Python does not check types; it checks for the presence of methods or attributes.

```python
class Dog:
    def speak(self):
        print("Woof")

class Cat:
    def speak(self):
        print("Meow")

class Car:
    def speak(self):
        print("Honk")

def make_it_speak(obj):
    # Doesn't care about the class, only that it has a speak() method
    obj.speak()

make_it_speak(Dog())  # Woof
make_it_speak(Car())  # Honk
```

This flexibility is central to Python's design. Use it to write generic, reusable code.

---

## Abstract Base Classes (ABCs)

Sometimes you want to enforce that a class *must* implement certain methods. Python's `abc` module provides a way to define Abstract Base Classes.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2
    
    # Missing perimeter()!

# c = Circle(5)  # TypeError: Can't instantiate abstract class Circle with abstract method perimeter
```

### `ABC` Internals
ABCs use a metaclass (`ABCMeta`) to register subclasses. When you call `isinstance(obj, MyABC)`, it checks if the object's class has been registered (either explicitly or via inheritance).

---

## Protocols (Static Duck Typing)

Introduced in Python 3.8 (PEP 544), `typing.Protocol` allows you to define structural types for static type checkers (mypy), bridging the gap between dynamic duck typing and static enforcement.

```python
from typing import Protocol

class Speakable(Protocol):
    def speak(self) -> None:
        ...

def make_it_speak(obj: Speakable):
    obj.speak()

class Dog:
    def speak(self):
        print("Woof")

# This passes static type checking, even though Dog doesn't inherit from Speakable
make_it_speak(Dog())
```

---

## Operator Overloading

Polymorphism also applies to operators. By implementing special methods, your classes can interact with standard Python operators (`+`, `-`, `len()`).

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)

v1 = Vector(1, 2)
v2 = Vector(3, 4)
v3 = v1 + v2  # Polymorphism in action: + works on Vectors
```

See [Special Methods](05_special_methods.md) for more.
