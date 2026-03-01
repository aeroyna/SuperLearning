# Class Decorators

Class decorators operate on a class. They receive the class object and typically return a modified class or a new class.

## Syntax

```python
@my_class_decorator
class MyClass:
    pass

# Equivalent to:
MyClass = my_class_decorator(MyClass)
```

## Use Case: Adding Methods

Dynamically adding functionality to a class.

```python
def add_str_method(cls):
    def __str__(self):
        return f"Instance of {self.__class__.__name__}"
    
    cls.__str__ = __str__
    return cls

@add_str_method
class Person:
    pass

p = Person()
print(p)  # "Instance of Person"
```

## Use Case: Registry (Singleton)

We saw this with metaclasses, but decorators can also do it simpler.

```python
instances = {}

def singleton(cls):
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class Database:
    def __init__(self):
        print("Loading database...")

d1 = Database()
d2 = Database()
# "Loading database..." prints once. d1 is d2.
```

## Class vs Metaclass?
*   **Decorator**: Modifies the class object *after* it's created. Easier to understand.
*   **Metaclass**: Intercepts the *creation* of the class. More powerful (can change inheritance, slots), but harder to debug.

Prefer decorators unless you strictly need to affect the creation process itself.
