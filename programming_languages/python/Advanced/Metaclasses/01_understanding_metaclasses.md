# Understanding Metaclasses

Metaclasses are often called "classes of classes". They control how classes are created, just as classes control how instances are created.

> *"Metaclasses are deeper magic than 99% of users should ever worry about. If you wonder whether you need them, you don't."* — Tim Peters

## What is a Class?

In Python, classes are themselves objects. When you define a class, Python executes the class body and creates a class object.

```python
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
```

`type` is the default metaclass for all classes in Python.

## One Line Class Creation

Because classes are objects, you can create them dynamically using `type()`:

`type(name, bases, dict)`

*   `name`: Name of the class (string)
*   `bases`: Tuple of parent classes
*   `dict`: Dictionary of class attributes/methods

```python
# Regular definition
class Foo:
    bar = True

# Equivalent dynamic definition
Foo = type('Foo', (), {'bar': True})
```

## The Metaclass Chain

1.  **Instance**: Created by Class. `type(obj)` is `Class`.
2.  **Class**: Created by Metaclass. `type(Class)` is `Metaclass`.
3.  **Metaclass**: Inherits from `type`.

```
Instance (obj) ──> Class (MyClass) ──> Metaclass (type)
```

## Why use Metaclasses?
They allow you to intercept class creation.
1.  **Registration**: Automatically register plugins.
2.  **Validation**: Ensure classes adhere to rules (e.g., must define specific methods).
3.  **Modification**: Automatically add methods or change attributes.

Most use cases for metaclasses (like decoration) can now be solved with **class decorators** or `__init_subclass__`, which are simpler.
