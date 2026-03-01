# Architecture Patterns

Common architectural patterns for building scalable, maintainable distributed systems.

---

## Topics in This Section

- [7.1 Monolithic vs Microservices](01_monolithic_vs_microservices.md)
- [7.2 Event-Driven Architecture](02_event_driven_architecture.md)
- [7.3 CQRS Pattern](03_cqrs.md)
- [7.4 Saga Pattern](04_saga_pattern.md)
- [7.5 Circuit Breaker Pattern](05_circuit_breaker.md)

---

## Pattern Overview

| Pattern | Problem Solved | Trade-offs |
|---------|---------------|------------|
| Microservices | Monolith scaling/deployment | Complexity, network calls |
| Event-Driven | Coupling, sync bottlenecks | Eventual consistency |
| CQRS | Read/write scaling mismatch | Data sync complexity |
| Saga | Distributed transactions | Complex error handling |
| Circuit Breaker | Cascade failures | Added latency check |

---

## When to Use Each Pattern

```
Monolith vs Microservices:
├── Small team, simple domain → Monolith
└── Large team, complex domain → Microservices

Synchronous vs Event-Driven:
├── Need immediate response → Synchronous
└── Can process async, need decoupling → Event-Driven

CQRS:
├── Read/write patterns similar → Skip
└── Heavy reads, complex queries → Use CQRS

Saga:
├── Single service transaction → Skip
└── Multi-service, need consistency → Use Saga
```

---

## Architecture Decision Guide

```
Starting a new project?
│
├── MVP/Startup phase
│   └── Start with Monolith (simpler)
│
├── Growing team (>20 engineers)?
│   └── Consider Microservices
│
├── High read:write ratio?
│   └── Consider CQRS
│
├── Need real-time updates?
│   └── Event-Driven Architecture
│
├── Distributed transactions?
│   └── Saga Pattern
│
└── Service dependencies?
    └── Circuit Breaker Pattern
```
