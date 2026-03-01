# Creational Patterns

Creational patterns deal with object creation mechanisms.

## Singleton
Ensures a class has only one instance.
See [Advanced/Metaclasses](../Metaclasses/02_custom_metaclasses.md) or [Intermediate/Decorators](../Decorators/02_class_decorators.md).

## Factory Method
Defines an interface for creating objects but let subclasses alter the type of objects created.

```python
class Animal:
    def speak(self): pass

class Dog(Animal):
    def speak(self): return "Woof"

class Cat(Animal):
    def speak(self): return "Meow"

class AnimalFactory:
    def create_animal(self, kind) -> Animal:
        if kind == "dog": return Dog()
        if kind == "cat": return Cat()
        raise ValueError("Unknown animal")

factory = AnimalFactory()
pet = factory.create_animal("dog")
```

## Builder
Constructs complex objects step by step.

```python
class Car:
    def __init__(self):
        self.parts = []

    def add(self, part):
        self.parts.append(part)

class CarBuilder:
    def __init__(self):
        self.car = Car()

    def add_engine(self):
        self.car.add("Engine")
        return self

    def add_wheels(self):
        self.car.add("Wheels")
        return self

    def build(self):
        return self.car

car = CarBuilder().add_engine().add_wheels().build()
```
