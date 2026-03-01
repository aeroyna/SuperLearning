# Inheritance

Inheritance allows a class (child) to derive attributes and methods from another class (parent), promoting code reuse and logical hierarchy. Python supports both single and multiple inheritance.

## Basic Inheritance

To inherit from a class, pass the parent class as an argument in the class definition.

```python
class Animal:
    def speak(self):
        return "Generic animal sound"

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    pass  # Inherits speak() from Animal directly

d = Dog()
print(d.speak())  # "Woof!"
```

---

## `super()`

The `super()` built-in function returns a **proxy object** that delegates method calls to a parent or sibling class. It is essential for correctly initializing parent classes and handling multiple inheritance.

### Common Usage
```python
class Person:
    def __init__(self, name):
        self.name = name

class Employee(Person):
    def __init__(self, name, emp_id):
        # Delegates to Person.__init__
        super().__init__(name)
        self.emp_id = emp_id
```

### Under the Hood
`super()` converts method calls into a search through the class's **MRO (Method Resolution Order)**. It does *not* simply call the parent.

```python
# Equivalent to:
super(Employee, self).__init__(name)
```

---

## Multiple Inheritance

Python allows a class to inherit from multiple parents.

```python
class Flyer:
    def fly(self):
        print("Flying!")

class Swimmer:
    def swim(self):
        print("Swimming!")

class Duck(Flyer, Swimmer):
    pass

d = Duck()
d.fly()
d.swim()
```

### The Diamond Problem and MRO
If multiple parent classes define the same method, Python must decide which one to call. This is determined by the **Method Resolution Order (MRO)**.

```python
class A:
    def show(self): print("A")

class B(A):
    def show(self): print("B")

class C(A):
    def show(self): print("C")

class D(B, C):
    pass

d = D()
d.show()  # Output: "B"
```

### C3 Linearization Algorithm
Python 2.3+ uses the C3 Linearization algorithm to determine MRO. It ensures:
1.  Children precede parents.
2.  The order of parents in the class definition is preserved.
3.  Each class appears only once in the MRO.

You can inspect the MRO:
```python
print(D.mro())
# [<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>]
```

If you construct a hierarchy that violates C3 (e.g., recursive cross-dependencies), Python raises a `TypeError`.

---

## Mixins

Mixins are classes designed to provide specific functionality to other classes via multiple inheritance, but are not intended to stand alone (e.g., `JsonSerializableMixin`).

```python
class JsonMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class Product(JsonMixin):
    def __init__(self, name, price):
        self.name = name
        self.price = price

p = Product("Laptop", 999)
print(p.to_json())
```

---

## `__init_subclass__` (Python 3.6+)

A cleaner alternative to metaclasses for simple class customization. It is called whenever a class inherits from the defining class.

```python
class Registry:
    plugins = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.plugins.append(cls)

class PluginA(Registry):
    pass

class PluginB(Registry):
    pass

print(Registry.plugins)  # [<class 'PluginA'>, <class 'PluginB'>]
```

---

## Best Practices
1.  **Prefer Composition over Inheritance**: If relationship is "has-a" rather than "is-a", use composition.
2.  **Avoid Deep Hierarchies**: They become hard to debug.
3.  **Use Mixins for Shared Behavior**: Keep inheritance trees flat.
