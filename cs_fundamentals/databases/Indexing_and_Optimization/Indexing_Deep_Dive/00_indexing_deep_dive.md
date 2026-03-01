# Indexing Deep Dive

## Introduction

Database indexes are specialized data structures that improve the speed of data retrieval operations at the cost of additional storage space and slower writes. Understanding index internals is crucial for designing efficient database systems and optimizing query performance.

## Why Indexing Matters

```
Without Index (Full Table Scan):
┌─────────────────────────────────────────────────────────────┐
│  Query: SELECT * FROM users WHERE email = 'john@example.com' │
├─────────────────────────────────────────────────────────────┤
│  Table: 1,000,000 rows                                       │
│  Time Complexity: O(n)                                       │
│  Disk I/O: ~1,000,000 row reads                             │
│  Estimated Time: 5-10 seconds                               │
└─────────────────────────────────────────────────────────────┘

With B-Tree Index:
┌─────────────────────────────────────────────────────────────┐
│  Query: SELECT * FROM users WHERE email = 'john@example.com' │
├─────────────────────────────────────────────────────────────┤
│  Index Height: 4 levels                                      │
│  Time Complexity: O(log n)                                   │
│  Disk I/O: 4-5 page reads                                   │
│  Estimated Time: <10 milliseconds                           │
└─────────────────────────────────────────────────────────────┘
```

## Index Data Structures Overview

### Comparison Matrix

| Data Structure | Read | Write | Range | Space | Best For |
|---------------|------|-------|-------|-------|----------|
| B-Tree | O(log n) | O(log n) | Excellent | Moderate | General purpose, OLTP |
| LSM Tree | O(log n) | O(1)* | Good | Higher | Write-heavy workloads |
| Hash Index | O(1) | O(1) | None | Low | Exact match lookups |
| Bloom Filter | O(k) | O(k) | None | Very Low | Membership testing |
| Skip List | O(log n) | O(log n) | Good | Higher | In-memory, concurrent |
| Inverted Index | O(1) | O(n) | N/A | High | Full-text search |

*Amortized

## Index Architecture Fundamentals

### Page-Based Storage

```
┌─────────────────────────────────────────────────────────────┐
│                    DISK PAGE (8KB typical)                   │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    Page Header                        │  │
│  │  - Page ID, LSN, Checksum, Free Space Pointer        │  │
│  └──────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    Item Pointers                      │  │
│  │  [Ptr1][Ptr2][Ptr3]...                               │  │
│  └──────────────────────────────────────────────────────┘  │
│                         ↓ Free Space ↓                      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    Tuple Data                         │  │
│  │  ...packed from bottom up                            │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Index Entry Structure

```
Index Entry Format:
┌────────────────────────────────────────────────────────┐
│  Key Value  │  Pointer/TID  │  Optional Metadata      │
├────────────────────────────────────────────────────────┤
│  "alice"    │  (page:42,    │  visibility info,       │
│             │   slot:7)     │  include columns        │
└────────────────────────────────────────────────────────┘

Composite Key Entry:
┌────────────────────────────────────────────────────────┐
│  Key1    │  Key2    │  Key3    │  TID           │
├────────────────────────────────────────────────────────┤
│  "USA"   │  "NY"    │  10001   │  (page:5,slot:3)│
└────────────────────────────────────────────────────────┘
```

## Index Types by Database

### PostgreSQL Index Types

```sql
-- B-Tree (default, most common)
CREATE INDEX idx_users_email ON users (email);

-- Hash (equality only)
CREATE INDEX idx_users_id_hash ON users USING hash (id);

-- GIN (arrays, full-text, JSONB)
CREATE INDEX idx_posts_tags ON posts USING gin (tags);
CREATE INDEX idx_docs_content ON documents USING gin (to_tsvector('english', content));

-- GiST (geometric, range types)
CREATE INDEX idx_locations_coords ON locations USING gist (coordinates);

-- BRIN (large sequential data)
CREATE INDEX idx_logs_timestamp ON logs USING brin (created_at);

-- SP-GiST (space-partitioned)
CREATE INDEX idx_points ON points USING spgist (location);
```

### MySQL/InnoDB Index Types

```sql
-- B+Tree (clustered and secondary)
CREATE INDEX idx_users_name ON users (last_name, first_name);

-- Full-text
CREATE FULLTEXT INDEX idx_articles_content ON articles (title, body);

-- Spatial (R-Tree)
CREATE SPATIAL INDEX idx_geo ON locations (geometry);

-- Hash (Memory engine only)
CREATE INDEX idx_cache_key ON cache_table (cache_key) USING HASH;
```

## Index Selection Strategy

### Decision Tree

```
                    ┌─────────────────────┐
                    │  What type of       │
                    │  queries do you     │
                    │  need to support?   │
                    └─────────┬───────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ Equality Only │    │ Range Queries │    │ Full-Text     │
│               │    │               │    │ Search        │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ Hash Index    │    │ B-Tree Index  │    │ Inverted      │
│ (if supported)│    │               │    │ Index (GIN)   │
└───────────────┘    └───────┬───────┘    └───────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
       ┌───────────┐  ┌───────────┐  ┌───────────┐
       │ Small     │  │ Large     │  │ Write     │
       │ Dataset   │  │ Ordered   │  │ Heavy     │
       │           │  │ Dataset   │  │           │
       └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
             │              │              │
             ▼              ▼              ▼
       ┌───────────┐  ┌───────────┐  ┌───────────┐
       │ Standard  │  │ BRIN      │  │ LSM Tree  │
       │ B-Tree    │  │ Index     │  │ (if avail)│
       └───────────┘  └───────────┘  └───────────┘
```

## Cost Analysis

### Index Overhead Calculation

```
Storage Overhead:
┌────────────────────────────────────────────────────────────┐
│  Table Size: 10 GB (1,000,000 rows)                        │
│  Average Row Size: 10 KB                                   │
├────────────────────────────────────────────────────────────┤
│  B-Tree Index on INT column (4 bytes):                     │
│    Entry Size: 4 (key) + 6 (TID) + 4 (overhead) = 14 bytes│
│    Entries: 1,000,000                                      │
│    Raw Size: 14 MB                                         │
│    With B-Tree overhead (~30%): ~18 MB                     │
│    Ratio: 0.18% of table size                              │
├────────────────────────────────────────────────────────────┤
│  B-Tree Index on VARCHAR(100) column:                      │
│    Average Key Size: 50 bytes                              │
│    Entry Size: 50 + 6 + 4 = 60 bytes                      │
│    With overhead: ~78 MB                                   │
│    Ratio: 0.78% of table size                              │
├────────────────────────────────────────────────────────────┤
│  Composite Index (3 columns):                              │
│    Could be 150-300 MB depending on column types           │
│    Ratio: 1.5-3% of table size                             │
└────────────────────────────────────────────────────────────┘
```

### Write Amplification

```
Write Operation Impact:
┌─────────────────────────────────────────────────────────────┐
│  INSERT with 0 indexes:                                      │
│    Operations: 1 table write                                 │
│    I/O: 1 page write (amortized)                            │
├─────────────────────────────────────────────────────────────┤
│  INSERT with 5 B-Tree indexes:                               │
│    Operations: 1 table + 5 index writes                      │
│    I/O: Up to 6 page writes                                  │
│    + Potential page splits in each index                     │
│    + WAL entries for each modification                       │
├─────────────────────────────────────────────────────────────┤
│  Write Amplification Factor: ~6-10x with 5 indexes          │
└─────────────────────────────────────────────────────────────┘
```

## Topics Covered

1. **[B-Tree Internals](01_btree_internals.md)** - Structure, operations, and variations
2. **[LSM Trees](02_lsm_trees.md)** - Log-structured merge trees for write optimization
3. **[Bloom Filters](03_bloom_filters.md)** - Probabilistic membership testing
4. **[Skip Lists](04_skip_lists.md)** - Probabilistic balanced structures
5. **[Inverted Indexes](05_inverted_indexes.md)** - Full-text search fundamentals

## Performance Guidelines

### Index Maintenance Best Practices

```sql
-- PostgreSQL: Monitor index bloat
SELECT
    schemaname || '.' || relname AS table_name,
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- MySQL: Check index usage
SELECT
    object_schema,
    object_name,
    index_name,
    count_star AS total_accesses,
    count_read,
    count_write
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE index_name IS NOT NULL
ORDER BY count_star DESC;
```

### When NOT to Index

1. **Small tables** (<1000 rows) - Full scan often faster
2. **Low selectivity columns** - Gender, boolean flags
3. **Frequently updated columns** - High write overhead
4. **Wide columns** - Large index size, less effective
5. **Expressions used rarely** - Storage waste

## Key Takeaways

- Index selection depends on query patterns, not just data types
- Every index adds write overhead - choose wisely
- Composite indexes require careful column ordering
- Regular monitoring prevents index bloat and unused indexes
- Different storage engines support different index types
