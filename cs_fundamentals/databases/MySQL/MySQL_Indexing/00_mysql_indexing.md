# MySQL Indexing

## Overview

Indexes are critical data structures that dramatically improve query performance in MySQL. Understanding how indexes work, when to use them, and how to optimize them is essential for database performance tuning.

This section covers:

1. **[B-Tree Indexes](01_btree_indexes.md)** - The default and most common index type
2. **[Hash Indexes](02_hash_indexes.md)** - Memory engine hash indexes
3. **[Full-Text Indexes](03_fulltext_indexes.md)** - Text search capabilities
4. **[Covering Indexes](04_covering_indexes.md)** - Index-only queries
5. **[Index Optimization](05_index_optimization.md)** - Best practices and tuning

---

## Why Indexes Matter

```
Without Index:                    With Index:
┌───────────────────────────┐    ┌───────────────────────────┐
│ SELECT * FROM users       │    │ SELECT * FROM users       │
│ WHERE email = 'a@b.com'   │    │ WHERE email = 'a@b.com'   │
│                           │    │                           │
│ Full Table Scan           │    │ Index Lookup              │
│ ┌─────────────────────┐   │    │ ┌─────────────────────┐   │
│ │ Row 1 ← check       │   │    │ │ B-Tree Index        │   │
│ │ Row 2 ← check       │   │    │ │ 'a@b.com' → Row 5   │   │
│ │ Row 3 ← check       │   │    │ └─────────────────────┘   │
│ │ Row 4 ← check       │   │    │          │               │
│ │ Row 5 ← FOUND!      │   │    │          ▼               │
│ │ Row 6 ← check       │   │    │ Direct access to Row 5   │
│ │ ...1M more rows     │   │    │                           │
│ └─────────────────────┘   │    │ O(log n) vs O(n)         │
│                           │    │                           │
│ Time: O(n) - 1M checks    │    │ Time: O(log n) - ~20 ops │
└───────────────────────────┘    └───────────────────────────┘
```

---

## Index Types in MySQL

| Index Type | Storage Engine | Use Case |
|------------|----------------|----------|
| B-Tree | InnoDB, MyISAM | Default, most queries |
| Hash | Memory, NDB | Exact match only |
| Full-Text | InnoDB, MyISAM | Text search |
| Spatial (R-Tree) | InnoDB, MyISAM | Geographic data |

---

## InnoDB Index Architecture

### Clustered Index

```
┌─────────────────────────────────────────────────────────────┐
│                    CLUSTERED INDEX                           │
│        (Primary Key = Physical Data Organization)            │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  Root Node                           │    │
│  │         [10 | 50 | 100 | 500 | 1000]                │    │
│  └───────────────────────┬─────────────────────────────┘    │
│              ┌───────────┼───────────┐                       │
│              ▼           ▼           ▼                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │ Leaf: 1-49  │ │ Leaf: 50-99 │ │ Leaf: 100+  │            │
│  │ [FULL ROWS] │ │ [FULL ROWS] │ │ [FULL ROWS] │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
│                                                              │
│  Leaf pages contain complete row data!                       │
└─────────────────────────────────────────────────────────────┘
```

### Secondary Index

```
┌─────────────────────────────────────────────────────────────┐
│                   SECONDARY INDEX                            │
│        (Index on non-primary key columns)                    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  Root Node                           │    │
│  │       [alice | bob | carol | dave | eve]            │    │
│  └───────────────────────┬─────────────────────────────┘    │
│              ┌───────────┼───────────┐                       │
│              ▼           ▼           ▼                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │ alice → PK5 │ │ bob → PK2   │ │ carol → PK8 │            │
│  │ amy → PK12  │ │ ben → PK7   │ │ cathy → PK3 │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
│                        │                                     │
│                        ▼ (lookup by PK)                      │
│               ┌─────────────────┐                            │
│               │ Clustered Index │                            │
│               │ PK2 → Full Row  │                            │
│               └─────────────────┘                            │
│                                                              │
│  Secondary index stores: indexed columns + primary key       │
└─────────────────────────────────────────────────────────────┘
```

---

## Quick Reference: Index Commands

```sql
-- Create index
CREATE INDEX idx_name ON table_name (column1, column2);
CREATE UNIQUE INDEX idx_email ON users (email);

-- Show indexes
SHOW INDEX FROM table_name;

-- Drop index
DROP INDEX idx_name ON table_name;
ALTER TABLE table_name DROP INDEX idx_name;

-- Analyze index usage
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE status = 'active';

-- Index statistics
ANALYZE TABLE table_name;
SHOW TABLE STATUS LIKE 'table_name';
```

---

## Learning Path

1. Start with B-Tree indexes - the foundation
2. Understand when hash indexes apply
3. Learn full-text for search features
4. Master covering indexes for performance
5. Apply optimization strategies
