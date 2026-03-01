# Structural Design Patterns

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

---

## Overview

| Pattern | Purpose | Key Benefit |
|---------|---------|-------------|
| [Adapter](01_adapter.md) | Convert interface | Legacy integration |
| [Decorator](02_decorator.md) | Add behavior dynamically | Flexible extension |
| [Facade](03_facade.md) | Simplify complex subsystem | Easy-to-use API |
| [Proxy](04_proxy.md) | Control access | Lazy loading, security |
| [Composite](05_composite.md) | Tree structures | Uniform treatment |

---

## Quick Reference

### Adapter
```python
class LegacyPrinter:
    def print_document(self, text):
        print(f"[Legacy] {text}")

class ModernPrinterAdapter(Printer):
    def __init__(self, legacy: LegacyPrinter):
        self.legacy = legacy

    def print(self, document):
        self.legacy.print_document(document.text)
```

### Decorator
```python
class LoggingDecorator(Service):
    def __init__(self, wrapped: Service):
        self.wrapped = wrapped

    def execute(self):
        print("Before")
        result = self.wrapped.execute()
        print("After")
        return result
```

### Facade
```python
class OrderFacade:
    def __init__(self):
        self.inventory = InventorySystem()
        self.payment = PaymentSystem()
        self.shipping = ShippingSystem()

    def place_order(self, order):
        self.inventory.reserve(order.items)
        self.payment.charge(order.total)
        self.shipping.schedule(order)
```

### Proxy
```python
class ImageProxy(Image):
    def __init__(self, filename):
        self.filename = filename
        self._real_image = None

    def display(self):
        if self._real_image is None:
            self._real_image = RealImage(self.filename)
        self._real_image.display()
```

### Composite
```python
class FileSystemComponent(ABC):
    @abstractmethod
    def get_size(self) -> int: pass

class File(FileSystemComponent):
    def get_size(self) -> int:
        return self.size

class Directory(FileSystemComponent):
    def get_size(self) -> int:
        return sum(child.get_size() for child in self.children)
```

---

## Choosing the Right Pattern

```
Need to integrate incompatible interfaces?
└── Use Adapter

Need to add responsibilities dynamically?
└── Use Decorator

Need to simplify complex subsystem?
└── Use Facade

Need to control access to an object?
└── Use Proxy

Need to represent part-whole hierarchies?
└── Use Composite
```

---

## Pattern Combinations

| Combination | Use Case |
|-------------|----------|
| Adapter + Facade | Simplify legacy system integration |
| Decorator + Proxy | Add behavior + control access |
| Composite + Decorator | Decorate tree structures |

---

**Tags**: #design-patterns #structural #overview
