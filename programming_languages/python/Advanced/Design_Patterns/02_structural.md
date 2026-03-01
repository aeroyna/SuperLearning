# Structural Patterns

Structural patterns deal with object composition.

## Adapter
Allows incompatible interfaces to work together.

```python
class OldSystem:
    def specific_request(self):
        return "Old"

class Adapter:
    def __init__(self, old_system):
        self.old_system = old_system

    def request(self):
        # Translate to new interface
        return f"Adapted: {self.old_system.specific_request()}"
```

## Decorator
Adds behavior dynamically.
See [Intermediate/Decorators](../../Intermediate/Decorators/01_function_decorators.md).

## Proxy
Controls access to an object.

```python
class RealService:
    def request(self):
        return "Done"

class ProxyService:
    def __init__(self):
        self.real_service = RealService()

    def request(self, user):
        if user != "admin":
            raise PermissionError("Forbidden")
        return self.real_service.request()
```
