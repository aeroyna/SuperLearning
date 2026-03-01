# Class vs Instance

Understanding the distinction between class-level and instance-level data is crucial for avoiding common bugs in Python.

## Attributes

### Instance Attributes
Bounded to a specific object. Unique to that instance. Defined inside `__init__` using `self`.

```python
class Dog:
    def __init__(self, name):
        self.name = name

d1 = Dog("Fido")
d2 = Dog("Spot")
# d1.name != d2.name
```

### Class Attributes
Shared by **all** instances of the class. Defined directly in the class body.

```python
class Dog:
    species = "Canis famillaris"  # Class attribute

    def __init__(self, name):
        self.name = name

d1 = Dog("Fido")
d2 = Dog("Spot")
print(d1.species)  # "Canis famillaris"
print(d2.species)  # "Canis famillaris"
```

### The Mutable Class Attribute Pitfall
Never use mutable objects (lists, dicts) as class attributes unless you explicitly want them shared across all instances.

```python
class BadDog:
    tricks = []  # SHARED by all dogs!

    def add_trick(self, trick):
        self.tricks.append(trick)

d1 = BadDog()
d2 = BadDog()

d1.add_trick("Roll over")
print(d2.tricks)  # ["Roll over"] - Oops! d2 knows the trick too.
```

**Fix:** Initialize in `__init__`.

---

## Methods

### Instance Methods
The default. Receives `self` (the instance) as the first argument. Can access instance and class state.

```python
class MyClass:
    def instance_method(self):
        return "I can access " + str(self)
```

### Class Methods (`@classmethod`)
Receives `cls` (the class) as the first argument. Can only access class state.
**Use case:** Alternative constructors (factory patterns).

```python
class Date:
    def __init__(self, day, month, year):
        self.day = day
        self.month = month
        self.year = year

    @classmethod
    def from_string(cls, date_str):
        day, month, year = map(int, date_str.split('-'))
        return cls(day, month, year)

d = Date.from_string("12-05-2023")
```

### Static Methods (`@staticmethod`)
Receives neither `self` nor `cls`. Just a regular function inside a class namespace.
**Use case:** Utility functions related to the class but independent of state.

```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b
```

---

## Summary Table

| Type | Decorator | First Arg | Can Modify Instance? | Can Modify Class? |
|------|-----------|-----------|----------------------|-------------------|
| **Instance** | None | `self` | Yes | Yes (via `self.__class__`) |
| **Class** | `@classmethod` | `cls` | No | Yes |
| **Static** | `@staticmethod` | None | No | No |
