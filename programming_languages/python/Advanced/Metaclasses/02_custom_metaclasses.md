# Custom Metaclasses

To create a custom metaclass, you inherit from `type` and override `__new__` or `__init__`.

## The `__new__` Method

`__new__` is the first step of class creation. It allocates memory for the class object. It is a static method (special case) that receives the metaclass as the first argument.

```python
class Meta(type):
    def __new__(cls, name, bases, dct):
        print(f"Creating class {name}")
        x = super().__new__(cls, name, bases, dct)
        return x

class MyClass(metaclass=Meta):
    pass
# Output: Creating class MyClass
```

### Parameters
*   `cls`: The metaclass itself.
*   `name`: Name of the class being created.
*   `bases`: Tuple of parent classes.
*   `dct`: Class attributes (dictionary).

## Example: Enforcing Snake Case Attributes

Let's create a metaclass that raises an error if class attributes are not snake_case.

```python
class SnakeCaseMeta(type):
    def __new__(cls, name, bases, dct):
        for attr_name in dct:
            if not attr_name.startswith("__") and not name.islower():
                 # Check logic here (simplified)
                 pass
        
        return super().__new__(cls, name, bases, dct)
```

## The `Singleton` Pattern via Metaclass

A classic use case. Ensure a class only has one instance.

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def connect(self):
        pass

db1 = Database()
db2 = Database()
print(db1 is db2)  # True
```
Note: In `SingletonMeta`, we override `__call__`. This is called when the *class* is called (instantiated).
