# ER Modeling

## Overview

Entity-Relationship (ER) modeling is a technique for designing database schemas. It provides a visual representation of entities, their attributes, and the relationships between them.

## Topics Covered

1. **[Entities and Attributes](01_entities_and_attributes.md)** - Defining database objects
2. **[Relationships and Cardinality](02_relationships_and_cardinality.md)** - How entities relate
3. **[ER to Relational Mapping](03_er_to_relational_mapping.md)** - Converting diagrams to tables
4. **[Advanced ER Concepts](04_advanced_er_concepts.md)** - Weak entities, inheritance, aggregation

## ER Diagram Notation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ER DIAGRAM SYMBOLS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ENTITY (Rectangle)              ATTRIBUTE (Ellipse)                       │
│   ┌───────────────┐               ╭───────────╮                             │
│   │   Customer    │               │   name    │                             │
│   └───────────────┘               ╰───────────╯                             │
│                                                                              │
│   RELATIONSHIP (Diamond)          KEY ATTRIBUTE (Underlined)                │
│       ╱╲                          ╭───────────╮                             │
│      ╱  ╲                         │    id     │                             │
│     ╱ has ╲                       ╰─────┬─────╯                             │
│      ╲  ╱                               (underlined)                        │
│       ╲╱                                                                    │
│                                                                              │
│   CARDINALITY NOTATION                                                       │
│   ─────────────────────                                                      │
│   │ (one)                 ──│     One (mandatory)                           │
│   ○ (zero)                ──○     Zero or One (optional)                    │
│   ◄ or ─┤├─ (many)       ──┤├─   Many (mandatory)                          │
│                           ──○┤├─  Zero or Many (optional many)              │
│                                                                              │
│   EXAMPLE:                                                                   │
│   ┌──────────┐         ╱╲        ┌──────────┐                               │
│   │ Customer │───────╱    ╲──────│  Order   │                               │
│   └──────────┘      ╲ places ╱   └──────────┘                               │
│       │              ╲    ╱          ├┤                                     │
│      (1)              ╲╱            (N)                                     │
│                                                                              │
│   "One customer places many orders"                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Quick Example

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    E-COMMERCE ER DIAGRAM                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌────────────┐       places        ┌────────────┐      contains           │
│   │  Customer  │─────────────────────│   Order    │────────────────┐        │
│   │            │ 1                 N │            │ 1            N │        │
│   │ •id (PK)   │                     │ •id (PK)   │                │        │
│   │ •name      │                     │ •date      │                │        │
│   │ •email     │                     │ •total     │                │        │
│   │ •address   │                     │ •status    │                │        │
│   └────────────┘                     └────────────┘                │        │
│                                             │                      │        │
│                                             │ belongs_to           │        │
│                                             │ N                    │        │
│                                             │                      │        │
│                                      ┌──────┴─────┐                │        │
│                                      │ OrderItem  │◄───────────────┘        │
│                                      │            │ N                       │
│   ┌────────────┐      is_in         │ •quantity  │       1 ┌────────────┐  │
│   │  Category  │◄───────────────────│ •price     │─────────│  Product   │  │
│   │            │ 1                N └────────────┘         │            │  │
│   │ •id (PK)   │                                           │ •id (PK)   │  │
│   │ •name      │                                           │ •name      │  │
│   │ •parent_id │                                           │ •price     │  │
│   └────────────┘                                           │ •stock     │  │
│         │                                                  └────────────┘  │
│         └──── self-referencing (subcategories)                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

After completing this section, you will be able to:
- Identify entities and their attributes from requirements
- Determine relationship types and cardinalities
- Create ER diagrams using standard notation
- Convert ER diagrams to relational schemas
- Handle complex scenarios like inheritance and weak entities
- Use ER modeling as a communication tool with stakeholders
