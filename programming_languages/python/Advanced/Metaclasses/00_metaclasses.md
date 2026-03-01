# Metaclasses

Metaclasses are "classes of classes" — they define how classes behave. While rarely needed, understanding them reveals Python's deep object model.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Understanding Metaclasses**](01_understanding_metaclasses.md) | type, class creation |
| [**2. Custom Metaclasses**](02_custom_metaclasses.md) | Creating and using metaclasses |
| [**3. Metaclass Use Cases**](03_use_cases.md) | ORMs, APIs, validation |

---

## Quick Reference

### Everything is an Object
```python
class MyClass:
    pass

# MyClass is an object
type(MyClass)  # <class 'type'>

# 'type' is the default metaclass
isinstance(MyClass, type)  # True

# Even 'type' is an instance of itself
type(type)  # <class 'type'>
```

### Class Creation with type()
```python
# Normal class definition
class Dog:
    species = "Canis familiaris"
    def bark(self):
        return "Woof!"

# Equivalent using type()
Dog = type('Dog', (), {
    'species': 'Canis familiaris',
    'bark': lambda self: "Woof!"
})
```

### Basic Metaclass
```python
class Meta(type):
    def __new__(mcs, name, bases, namespace):
        print(f"Creating class: {name}")
        cls = super().__new__(mcs, name, bases, namespace)
        return cls

class MyClass(metaclass=Meta):
    pass
# Prints: Creating class: MyClass
```

---

## Common Patterns

### Singleton
```python
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    pass

db1 = Database()
db2 = Database()
db1 is db2  # True
```

### Automatic Registration
```python
class PluginMeta(type):
    plugins = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if name != 'Plugin':  # Don't register base class
            mcs.plugins[name] = cls
        return cls

class Plugin(metaclass=PluginMeta):
    pass

class MyPlugin(Plugin):
    pass

print(PluginMeta.plugins)  # {'MyPlugin': <class 'MyPlugin'>}
```

---

## When to Use Metaclasses

1. **Framework/Library Development** — Django ORM, SQLAlchemy
2. **Automatic Registration** — Plugin systems
3. **Validation** — Enforce class structure
4. **Modification** — Add methods/attributes automatically

**Tim Peters' Rule**: "Metaclasses are deeper magic than 99% of users should ever worry about."

---

## Alternatives to Metaclasses

- **Class Decorators** — Simpler for most modifications
- **`__init_subclass__`** — Hook into subclass creation (Python 3.6+)
- **Descriptors** — Control attribute access

```python
# __init_subclass__ alternative
class Plugin:
    plugins = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin.plugins[cls.__name__] = cls

class MyPlugin(Plugin):
    pass
```

---

## Next Steps
Start with [Understanding Metaclasses](01_understanding_metaclasses.md).
