# Schema Design

## Introduction

Good schema design is the foundation of a performant, maintainable database. It affects query performance, data integrity, storage efficiency, and application development complexity.

## Topics in This Section

1. **[Design Methodology](01_design_methodology.md)**
2. **[Common Schema Patterns](02_common_schema_patterns.md)**
3. **[Temporal Data Modeling](03_temporal_data_modeling.md)**
4. **[Multi-Tenancy Patterns](04_multi_tenancy_patterns.md)**

## Schema Design Principles

```
┌─────────────────────────────────────────────────────────────────┐
│              Core Design Principles                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. MODEL THE DOMAIN ACCURATELY                                 │
│     • Reflect real-world entities and relationships            │
│     • Use appropriate data types                                │
│     • Capture business rules in constraints                     │
│                                                                  │
│  2. DESIGN FOR ACCESS PATTERNS                                  │
│     • Understand how data will be queried                       │
│     • Optimize for common operations                            │
│     • Balance read vs write performance                         │
│                                                                  │
│  3. ENSURE DATA INTEGRITY                                       │
│     • Use primary keys and foreign keys                         │
│     • Apply appropriate constraints                             │
│     • Validate data at the database level                       │
│                                                                  │
│  4. PLAN FOR GROWTH                                             │
│     • Consider data volume growth                               │
│     • Design for horizontal scaling if needed                   │
│     • Avoid schema decisions that limit future options          │
│                                                                  │
│  5. KEEP IT SIMPLE                                              │
│     • Avoid over-engineering                                    │
│     • Use standard patterns when applicable                     │
│     • Document design decisions                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Design Process Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              Schema Design Workflow                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 1. REQUIREMENTS GATHERING                                │    │
│  │    • Business requirements                               │    │
│  │    • Data entities and relationships                     │    │
│  │    • Access patterns and queries                         │    │
│  │    • Volume and growth estimates                         │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 2. CONCEPTUAL DESIGN                                     │    │
│  │    • Entity-Relationship diagram                         │    │
│  │    • Identify entities and attributes                    │    │
│  │    • Define relationships and cardinality                │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 3. LOGICAL DESIGN                                        │    │
│  │    • Convert to tables and columns                       │    │
│  │    • Apply normalization rules                           │    │
│  │    • Define primary and foreign keys                     │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 4. PHYSICAL DESIGN                                       │    │
│  │    • Choose data types                                   │    │
│  │    • Design indexes                                      │    │
│  │    • Plan partitioning if needed                         │    │
│  │    • Consider denormalization for performance            │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 5. IMPLEMENTATION & ITERATION                            │    │
│  │    • Create schema                                       │    │
│  │    • Load test data                                      │    │
│  │    • Benchmark queries                                   │    │
│  │    • Refine based on results                             │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Data Type Selection

```
┌─────────────────────────────────────────────────────────────────┐
│              Choosing Data Types                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NUMERIC TYPES                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Use Case              Recommended Type                     │ │
│  │ ─────────────────────────────────────────────────────────  │ │
│  │ Small integers        SMALLINT (2 bytes, ±32K)            │ │
│  │ Regular integers      INTEGER (4 bytes, ±2B)              │ │
│  │ Large integers        BIGINT (8 bytes)                    │ │
│  │ Money/currency        DECIMAL(19,4) or NUMERIC            │ │
│  │ Floating point        DOUBLE PRECISION (avoid for money)  │ │
│  │ Auto-increment IDs    BIGINT (plan for growth)            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STRING TYPES                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Use Case              Recommended Type                     │ │
│  │ ─────────────────────────────────────────────────────────  │ │
│  │ Fixed length          CHAR(n) - padded                    │ │
│  │ Variable length       VARCHAR(n) - with limit             │ │
│  │ Unlimited text        TEXT                                │ │
│  │ Binary data           BYTEA / BLOB                        │ │
│  │ UUIDs                 UUID (native) or CHAR(36)           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATE/TIME TYPES                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Use Case              Recommended Type                     │ │
│  │ ─────────────────────────────────────────────────────────  │ │
│  │ Date only             DATE                                │ │
│  │ Time only             TIME                                │ │
│  │ Date + time (local)   TIMESTAMP                           │ │
│  │ Date + time (UTC)     TIMESTAMP WITH TIME ZONE            │ │
│  │ Duration              INTERVAL                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BEST PRACTICES                                                 │
│  • Use the smallest type that fits your data                   │
│  • Prefer native types over strings (e.g., UUID, INET)         │
│  • Always use TIMESTAMP WITH TIME ZONE for timestamps          │
│  • Use DECIMAL for money, never FLOAT                          │
└─────────────────────────────────────────────────────────────────┘
```

## Common Design Decisions

```
┌─────────────────────────────────────────────────────────────────┐
│              Key Design Decisions                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PRIMARY KEY CHOICE                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Auto-increment (SERIAL/IDENTITY):                         │ │
│  │ ✓ Simple, compact, ordered                                │ │
│  │ ✗ Predictable, can expose count                          │ │
│  │ ✗ Problematic for distributed systems                    │ │
│  │                                                             │ │
│  │ UUID:                                                       │ │
│  │ ✓ Globally unique, no coordination needed                 │ │
│  │ ✓ Safe to expose externally                               │ │
│  │ ✗ Larger (16 bytes), random = poor locality              │ │
│  │ Tip: Use UUIDv7 for time-ordered UUIDs                    │ │
│  │                                                             │ │
│  │ Natural key:                                                │ │
│  │ ✓ Meaningful, may eliminate joins                         │ │
│  │ ✗ Can change, often compound                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NORMALIZATION VS DENORMALIZATION                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Normalize when:                                             │ │
│  │ • Data integrity is critical                               │ │
│  │ • Write performance matters                                │ │
│  │ • Storage efficiency is important                          │ │
│  │                                                             │ │
│  │ Denormalize when:                                           │ │
│  │ • Read performance is critical                             │ │
│  │ • Joins are too expensive                                  │ │
│  │ • Data rarely changes                                      │ │
│  │                                                             │ │
│  │ Balance: Start normalized, denormalize where needed        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
