# PostgreSQL Indexing

## Overview

PostgreSQL offers one of the most sophisticated indexing systems among relational databases. Understanding these index types and when to use each is crucial for query optimization and database performance.

---

## What You'll Learn

### 1. B-Tree Internals
- B-Tree structure and algorithms
- Key ordering and duplicate handling
- When B-Tree is optimal
- B-Tree specific operations

### 2. Hash, GIN, and GiST Indexes
- Hash index use cases
- GIN for full-text and arrays
- GiST for geometric and range data
- Operator class selection

### 3. BRIN and Bloom Indexes
- Block Range Indexes for large tables
- Bloom filters for multi-column queries
- When to use specialized indexes

### 4. Partial and Expression Indexes
- Indexing subsets of data
- Computed value indexes
- Optimizing specific query patterns

### 5. Index-Only Scans and Covering Indexes
- Avoiding table access
- INCLUDE clause usage
- Visibility map optimization

---

## Index Types Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Index Types                            │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  B-TREE (Default)                                           │    │
│  │  • Equality and range queries (<, <=, =, >=, >)             │    │
│  │  • LIKE 'prefix%' patterns                                  │    │
│  │  • ORDER BY optimization                                    │    │
│  │  • Most common choice                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  HASH                                                        │    │
│  │  • Equality only (=)                                         │    │
│  │  • Smaller than B-tree for equality                          │    │
│  │  • No range support                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  GIN (Generalized Inverted Index)                           │    │
│  │  • Full-text search (tsvector)                               │    │
│  │  • Arrays, JSONB containment                                 │    │
│  │  • Multi-valued columns                                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  GiST (Generalized Search Tree)                              │    │
│  │  • Geometric data                                            │    │
│  │  • Range types                                               │    │
│  │  • Full-text (alternative to GIN)                            │    │
│  │  • Nearest-neighbor searches                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  BRIN (Block Range Index)                                    │    │
│  │  • Very large tables with natural ordering                   │    │
│  │  • Time-series data                                          │    │
│  │  • Minimal storage overhead                                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  SP-GiST (Space-Partitioned GiST)                            │    │
│  │  • Quadtrees, k-d trees                                      │    │
│  │  • Non-balanced data structures                              │    │
│  │  • Phone numbers, IP addresses                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference

```sql
-- Create different index types
CREATE INDEX idx_btree ON table USING btree (column);  -- Default
CREATE INDEX idx_hash ON table USING hash (column);
CREATE INDEX idx_gin ON table USING gin (column);
CREATE INDEX idx_gist ON table USING gist (column);
CREATE INDEX idx_brin ON table USING brin (column);
CREATE INDEX idx_spgist ON table USING spgist (column);

-- Partial index
CREATE INDEX idx_active ON users (email) WHERE active = true;

-- Expression index
CREATE INDEX idx_lower ON users (lower(email));

-- Covering index
CREATE INDEX idx_covering ON orders (user_id) INCLUDE (total, status);

-- Multi-column index
CREATE INDEX idx_multi ON orders (user_id, created_at DESC);
```

---

## Index Selection Guide

| Query Pattern | Recommended Index |
|---------------|-------------------|
| `column = value` | B-tree or Hash |
| `column < value` | B-tree |
| `column BETWEEN x AND y` | B-tree |
| `column LIKE 'prefix%'` | B-tree |
| `column LIKE '%pattern%'` | GIN (pg_trgm) |
| `array @> ARRAY[value]` | GIN |
| `jsonb @> '{...}'` | GIN |
| `tsvector @@ tsquery` | GIN or GiST |
| `range && range` | GiST |
| `point <-> point` | GiST |
| Natural ordering (time) | BRIN |
| Many columns, rare combo | Bloom |

---

## Section Files

| File | Topic |
|------|-------|
| [01_btree_internals.md](01_btree_internals.md) | B-Tree structure and optimization |
| [02_hash_gin_gist.md](02_hash_gin_gist.md) | Hash, GIN, and GiST indexes |
| [03_brin_and_bloom.md](03_brin_and_bloom.md) | BRIN and Bloom indexes |
| [04_partial_expression_indexes.md](04_partial_expression_indexes.md) | Partial and expression indexes |
| [05_index_only_scans.md](05_index_only_scans.md) | Index-only scans and covering indexes |

---

## Further Reading

- PostgreSQL Index Types documentation
- "The Art of PostgreSQL" - Index chapters
- PostgreSQL Query Planner documentation
