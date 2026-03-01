# Design Patterns in Python

Classic design patterns implemented in a Pythonic way.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Creational Patterns**](01_creational.md) | Object creation |
| [**2. Structural Patterns**](02_structural.md) | Object composition |
| [**3. Behavioral Patterns**](03_behavioral.md) | Object interaction |

---

## Quick Reference

### Singleton
```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Or using module
# mysingleton.py — module itself is singleton
```

### Factory
```python
def create_user(user_type):
    if user_type == "admin":
        return AdminUser()
    elif user_type == "guest":
        return GuestUser()
    else:
        return RegularUser()
```

### Strategy
```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount): pass

class CreditCard(PaymentStrategy):
    def pay(self, amount):
        return f"Paid ${amount} with credit card"

class PayPal(PaymentStrategy):
    def pay(self, amount):
        return f"Paid ${amount} with PayPal"

class ShoppingCart:
    def __init__(self, strategy: PaymentStrategy):
        self.strategy = strategy

    def checkout(self, amount):
        return self.strategy.pay(amount)
```

### Observer
```python
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        print(f"Received: {message}")
```

### Decorator (Pattern, not Python decorator)
```python
class Coffee:
    def cost(self): return 1.00

class MilkDecorator:
    def __init__(self, coffee):
        self._coffee = coffee

    def cost(self):
        return self._coffee.cost() + 0.50

coffee = MilkDecorator(Coffee())
print(coffee.cost())  # 1.50
```

---

## Python-Specific Patterns

### Borg (Shared State)
```python
class Borg:
    _shared_state = {}

    def __init__(self):
        self.__dict__ = self._shared_state
```

### Mixin
```python
class LoggerMixin:
    def log(self, message):
        print(f"[{self.__class__.__name__}] {message}")

class MyClass(LoggerMixin):
    def do_something(self):
        self.log("Doing something")
```

---

## Next Steps
Start with [Creational Patterns](01_creational.md).
