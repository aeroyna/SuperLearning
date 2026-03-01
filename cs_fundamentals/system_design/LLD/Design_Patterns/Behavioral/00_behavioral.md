# Behavioral Design Patterns

Behavioral patterns focus on communication and interaction between objects.

---

## Patterns in This Section

- [01. Strategy](01_strategy.md) - Interchangeable algorithms
- [02. Observer](02_observer.md) - Publish-subscribe
- [03. Command](03_command.md) - Encapsulate requests
- [04. State](04_state.md) - State-based behavior
- [05. Chain of Responsibility](05_chain_of_responsibility.md) - Pass requests along chain
- [06. Template Method](06_template_method.md) - Define skeleton algorithm

---

## Quick Comparison

| Pattern | When to Use | Example |
|---------|-------------|---------|
| Strategy | Multiple algorithms for same task | Sorting, pricing, validation |
| Observer | Notify multiple objects of changes | Event systems, UI updates |
| Command | Queue/log/undo operations | Editor actions, transactions |
| State | Object behavior depends on state | Vending machine, orders |
| Chain of Responsibility | Multiple handlers for request | Middleware, approval chains |
| Template Method | Common algorithm, varying steps | Data parsers, test frameworks |

---

## Decision Guide

```
Defining object behavior?
│
├── Multiple interchangeable algorithms?
│   └── Strategy
│
├── Need to notify multiple observers?
│   └── Observer
│
├── Need to queue/undo/log operations?
│   └── Command
│
├── Behavior depends on internal state?
│   └── State
│
├── Request processed by multiple handlers?
│   └── Chain of Responsibility
│
└── Same algorithm, different steps?
    └── Template Method
```
