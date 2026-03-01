# Data Models

## Overview

A **data model** defines how data is organized, stored, and manipulated. It provides the conceptual framework for representing real-world entities and their relationships within a database system.

## Topics Covered

1. **[Relational Model](01_relational_model.md)** - Tables, rows, columns, and SQL operations
2. **[Document Model](02_document_model.md)** - Flexible JSON/BSON documents
3. **[Key-Value Model](03_key_value_model.md)** - Simple key-based lookups
4. **[Graph Model](04_graph_model.md)** - Nodes and relationships
5. **[Wide-Column Model](05_wide_column_model.md)** - Column families for big data

## Data Model Comparison

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATA MODEL SPECTRUM                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Structured ◄──────────────────────────────────────────► Flexible          │
│                                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│   │Relational│  │Wide-Col  │  │ Document │  │  Graph   │  │Key-Value │     │
│   │          │  │          │  │          │  │          │  │          │     │
│   │ Tables   │  │ Columns  │  │  JSON    │  │  Nodes   │  │ Key:Val  │     │
│   │ Rows     │  │ Families │  │  Nested  │  │  Edges   │  │  Pairs   │     │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│                                                                              │
│   Schema-on-Write                                         Schema-on-Read    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Schema** | The structure that defines how data is organized |
| **Entity** | A real-world object represented in the database |
| **Relationship** | How entities connect to each other |
| **Query Language** | Method to retrieve and manipulate data |
| **Consistency Model** | Guarantees about data correctness |

## Learning Objectives

After completing this section, you will be able to:
- Understand the trade-offs between different data models
- Choose the appropriate data model for specific use cases
- Map real-world entities to database structures
- Understand schema design principles for each model
- Evaluate query capabilities and limitations of each model
