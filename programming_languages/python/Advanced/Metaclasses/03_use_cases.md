# Metaclass Use Cases

While metaclasses are powerful, they should be used sparingly. Here are legitimate use cases where they shine.

## 1. Automatic Registration (Plugins)

You want to maintain a registry of all subclasses of a particular base class (e.g., for a plugin system).

```python
class PluginMeta(type):
    registry = {}

    def __new__(cls, name, bases, attrs):
        new_class = super().__new__(cls, name, bases, attrs)
        # Register the class using its name (or a special id)
        if name != 'BasePlugin':
             cls.registry[name] = new_class
        return new_class

class BasePlugin(metaclass=PluginMeta):
    pass

class AudioPlugin(BasePlugin):
    pass

class VideoPlugin(BasePlugin):
    pass

print(PluginMeta.registry)
# {'AudioPlugin': <class '__main__.AudioPlugin'>, 'VideoPlugin': <class '__main__.VideoPlugin'>}
```

## 2. Interface Validation

Enforcing that subclasses define specific attributes or methods, similar to ABCs but with more control during creation time.

```python
class APIInterface(type):
    def __new__(cls, name, bases, attrs):
        if name != 'BaseAPI':
            if 'connect' not in attrs:
                raise TypeError(f"Class {name} must implement 'connect' method")
        return super().__new__(cls, name, bases, attrs)

class BaseAPI(metaclass=APIInterface):
    pass

# class TwitterAPI(BaseAPI):
#     pass
# TypeError: Class TwitterAPI must implement 'connect' method
```

## 3. ORM (Object Relational Mapping)

This is the most common real-world use case (e.g., Django Models, SQLAlchemy). Metaclasses map class attributes to database columns.

```python
class Field:
    pass

class IntegerField(Field):
    pass

class ModelMeta(type):
    def __new__(cls, name, bases, attrs):
        fields = {k: v for k, v in attrs.items() if isinstance(v, Field)}
        attrs['_fields'] = fields
        return super().__new__(cls, name, bases, attrs)

class User(metaclass=ModelMeta):
    id = IntegerField()
    age = IntegerField()

print(User._fields)
# {'id': <__main__.IntegerField object>, 'age': <__main__.IntegerField object>}
```

## Alternatives
Before using a metaclass, consider:
1.  **Class Decorators**: Can modify a class after creation.
2.  **`__init_subclass__`**: Simpler hook for subclass creation (Python 3.6+).

```python
# Registration using __init_subclass__
class Plugin:
    registry = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        cls.registry.append(cls)
```
