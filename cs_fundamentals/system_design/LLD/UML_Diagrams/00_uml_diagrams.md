# UML Diagrams

> "UML (Unified Modeling Language) is a standardized visual language for specifying, constructing, and documenting software systems."

## Overview

UML provides a standard way to visualize the design of a system. In LLD interviews, you'll primarily use:

| Diagram Type | Purpose | When to Use |
|--------------|---------|-------------|
| **Class Diagram** | Static structure | Always - core of LLD |
| **Sequence Diagram** | Object interactions | Complex workflows |
| **Use Case Diagram** | System functionality | Requirements phase |
| **State Diagram** | State transitions | State-dependent behavior |
| **Activity Diagram** | Process flow | Algorithms, workflows |

---

## Diagram Hierarchy

```
UML Diagrams
├── Structure Diagrams (Static)
│   ├── Class Diagram ★
│   ├── Object Diagram
│   ├── Component Diagram
│   └── Package Diagram
│
└── Behavior Diagrams (Dynamic)
    ├── Use Case Diagram ★
    ├── Sequence Diagram ★
    ├── Activity Diagram
    ├── State Diagram ★
    └── Communication Diagram
```

★ = Most important for LLD interviews

---

## Quick Reference

### Class Diagram Notation

```
┌─────────────────────────┐
│     <<interface>>       │   Abstract/Interface
│       IService          │
├─────────────────────────┤
│ + method(): ReturnType  │
└─────────────────────────┘
           △
           │ implements
┌─────────────────────────┐
│      ClassName          │   Concrete Class
├─────────────────────────┤
│ - privateField: Type    │   Attributes
│ # protectedField: Type  │
│ + publicField: Type     │
├─────────────────────────┤
│ + publicMethod(): Type  │   Methods
│ - privateMethod(): void │
└─────────────────────────┘
```

### Relationship Symbols

```
A ────────► B    Association (A uses B)
A ◆────────► B    Composition (A owns B, B dies with A)
A ◇────────► B    Aggregation (A has B, B can exist alone)
A ─────────△ B    Inheritance (A extends B)
A - - - - -△ B    Implementation (A implements B)
A - - - - -► B    Dependency (A depends on B)
```

### Multiplicity

```
1        Exactly one
0..1     Zero or one
*        Zero or more
1..*     One or more
n        Exactly n
n..m     Between n and m
```

---

## Interview Approach

### Step 1: Identify Entities
- What are the main objects in the system?
- What are their attributes and behaviors?

### Step 2: Define Relationships
- How do objects relate to each other?
- What are the cardinalities?

### Step 3: Draw Class Diagram
- Start with core classes
- Add relationships
- Include key methods

### Step 4: Add Sequence Diagrams
- Show important workflows
- Illustrate object interactions

---

## Tools for Drawing

### Whiteboard/Interview
- Keep it simple
- Focus on clarity over perfection
- Use standard notation

### Digital Tools
- [draw.io](https://draw.io) - Free, web-based
- [Lucidchart](https://lucidchart.com) - Feature-rich
- [PlantUML](https://plantuml.com) - Text-based
- [Mermaid](https://mermaid.js.org) - Markdown-friendly

---

## Related Topics

- [[01_class_diagrams|Class Diagrams]] - Detailed guide
- [[02_sequence_diagrams|Sequence Diagrams]] - Interaction flows
- [[03_use_case_diagrams|Use Case Diagrams]] - System functionality

---

**Tags**: #uml #diagrams #lld #modeling #design
