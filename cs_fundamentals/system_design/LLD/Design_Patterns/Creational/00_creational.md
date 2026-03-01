# Creational Design Patterns

Creational patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

---

## Patterns in This Section

- [01. Singleton](01_singleton.md) - Ensure single instance
- [02. Factory Method](02_factory.md) - Create without specifying class
- [03. Abstract Factory](03_abstract_factory.md) - Create families of objects
- [04. Builder](04_builder.md) - Construct complex objects step by step
- [05. Prototype](05_prototype.md) - Clone existing objects

---

## Quick Comparison

| Pattern | When to Use | Example |
|---------|-------------|---------|
| Singleton | Need exactly one instance | Logger, Config |
| Factory | Create one of many types | Document (PDF, Word, Excel) |
| Abstract Factory | Create families of related objects | UI Kit (Button, Menu, Dialog) |
| Builder | Complex object with many options | HTTP Request, SQL Query |
| Prototype | Clone is cheaper than create | Game objects, cached configs |

---

## Decision Guide

```
Creating objects?
│
├── Need exactly one instance?
│   └── Singleton
│
├── Creating one of many related types?
│   └── Factory Method
│
├── Creating families of related objects?
│   └── Abstract Factory
│
├── Object has many optional components?
│   └── Builder
│
└── Creating copies of existing object?
    └── Prototype
```
