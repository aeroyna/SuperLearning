# B-Tree Internals

## Learning Objectives
- Understand B-Tree structure in PostgreSQL
- Master B-Tree operations and algorithms
- Optimize B-Tree index usage
- Handle duplicate keys effectively

---

## 1. B-Tree Structure

### Anatomy of a B-Tree

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL B-Tree Structure                       │
│                                                                      │
│                         ┌───────────────┐                            │
│                         │   Meta Page   │  Page 0                    │
│                         │   (Root ptr)  │                            │
│                         └───────┬───────┘                            │
│                                 │                                    │
│                                 ▼                                    │
│                         ┌───────────────┐                            │
│                         │  Root Page    │                            │
│                         │ [30 | 60 | 90]│                            │
│                         └───┬───┬───┬───┘                            │
│                    ┌────────┘   │   └────────┐                       │
│                    ▼            ▼            ▼                       │
│              ┌──────────┐ ┌──────────┐ ┌──────────┐                  │
│              │[10|20|25]│ │[35|45|55]│ │[65|75|85]│  Internal Pages  │
│              └──┬─┬─┬───┘ └──┬─┬─┬───┘ └──┬─┬─┬───┘                  │
│                 │ │ │        │ │ │        │ │ │                      │
│                 ▼ ▼ ▼        ▼ ▼ ▼        ▼ ▼ ▼                      │
│              ┌─────────────────────────────────────┐                 │
│              │         Leaf Pages (Data)           │                 │
│              │  [1,2,3] [10,11] [20,21,22] ...     │                 │
│              │                                     │                 │
│              │  ◄────── Sibling Links ──────►      │                 │
│              └─────────────────────────────────────┘                 │
│                                                                      │
│  Key Properties:                                                     │
│  • Balanced: All leaf pages at same depth                           │
│  • Sorted: Keys ordered within and across pages                     │
│  • Linked: Leaf pages have sibling pointers                         │
│  • Dense: Leaf pages contain actual index entries                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Page Layout

```sql
-- B-Tree page structure (8KB default)
-- ┌────────────────────────────────────────────────────────────┐
-- │ Page Header (24 bytes)                                      │
-- │   - LSN, checksum, flags, lower, upper, special            │
-- ├────────────────────────────────────────────────────────────┤
-- │ Item ID Array (4 bytes per item)                            │
-- │   - Pointers to tuples within page                          │
-- ├────────────────────────────────────────────────────────────┤
-- │ Free Space                                                  │
-- ├────────────────────────────────────────────────────────────┤
-- │ Index Tuples (variable size)                                │
-- │   - Key value(s) + TID (heap tuple ID)                      │
-- ├────────────────────────────────────────────────────────────┤
-- │ Special Space (16 bytes for B-Tree)                         │
-- │   - Left/right sibling pointers, flags                      │
-- └────────────────────────────────────────────────────────────┘

-- View B-Tree structure with pageinspect
CREATE EXTENSION IF NOT EXISTS pageinspect;

-- Metapage info
SELECT * FROM bt_metap('idx_users_email');
-- magic, version, root, level, fastroot, fastlevel

-- Page statistics
SELECT * FROM bt_page_stats('idx_users_email', 1);
-- blkno, type, live_items, dead_items, avg_item_size, free_size
```

---

## 2. B-Tree Operations

### Search Operation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    B-Tree Search: Find key = 47                      │
│                                                                      │
│  Step 1: Start at root                                               │
│          ┌───────────────┐                                           │
│          │ [30 | 60 | 90]│  47 > 30, 47 < 60 → follow middle        │
│          └───────┬───────┘                                           │
│                  │                                                   │
│  Step 2: Internal page                                               │
│          ┌───────────────┐                                           │
│          │ [35 | 45 | 55]│  47 > 45, 47 < 55 → follow third          │
│          └───────┬───────┘                                           │
│                  │                                                   │
│  Step 3: Leaf page                                                   │
│          ┌───────────────┐                                           │
│          │ [46|47|48|49] │  Binary search → found at position 1     │
│          └───────────────┘                                           │
│                                                                      │
│  Complexity: O(log n) - typically 3-4 page reads for millions rows  │
└─────────────────────────────────────────────────────────────────────┘
```

### Range Scan

```sql
-- Range query optimization
SELECT * FROM users WHERE age BETWEEN 25 AND 35;

-- B-Tree execution:
-- 1. Search for start key (25) → O(log n)
-- 2. Follow leaf sibling pointers → O(k) where k = result count
-- 3. Stop when exceeding end key (35)

-- Bidirectional scan (ORDER BY DESC)
SELECT * FROM users WHERE age < 30 ORDER BY age DESC;
-- Starts from key 30, follows left sibling pointers
```

### Insert Operation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    B-Tree Insert: Add key = 42                       │
│                                                                      │
│  Step 1: Find leaf page (search)                                     │
│                                                                      │
│  Step 2: Insert if space available                                   │
│          ┌───────────────┐    ┌───────────────────┐                  │
│          │ [40 | 45 | 50]│ → │ [40 | 42 | 45 | 50]│                  │
│          └───────────────┘    └───────────────────┘                  │
│                                                                      │
│  Step 3: If page full, split                                         │
│          ┌───────────────┐                                           │
│          │ Full Page     │                                           │
│          └───────────────┘                                           │
│                 │                                                    │
│                 ▼ Split                                              │
│          ┌──────────┐  ┌──────────┐                                  │
│          │ Left Half│  │Right Half│                                  │
│          └──────────┘  └──────────┘                                  │
│                 ↑                                                    │
│          Middle key promoted to parent                               │
│          (May cascade splits up the tree)                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Duplicate Handling

### Duplicate Keys in B-Tree

```sql
-- PostgreSQL B-Tree handles duplicates via posting lists (v13+)
CREATE TABLE logs (
    id SERIAL PRIMARY KEY,
    level VARCHAR(10),
    message TEXT
);

CREATE INDEX idx_logs_level ON logs (level);

-- Many rows with same level value
INSERT INTO logs (level, message)
SELECT 'INFO', 'Log message ' || i
FROM generate_series(1, 100000) i;

-- Deduplication compresses duplicates
-- Key stored once, TIDs stored in posting list
```

### Deduplication (PostgreSQL 13+)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    B-Tree Deduplication                              │
│                                                                      │
│  Without Deduplication:                                              │
│  ┌──────────────────────────────────────────────────┐               │
│  │ [INFO,(1,1)] [INFO,(1,2)] [INFO,(1,3)] ...       │               │
│  │ Each duplicate = separate index entry            │               │
│  └──────────────────────────────────────────────────┘               │
│                                                                      │
│  With Deduplication:                                                 │
│  ┌──────────────────────────────────────────────────┐               │
│  │ [INFO, {(1,1), (1,2), (1,3), ...}]               │               │
│  │ Single key + posting list of TIDs               │               │
│  └──────────────────────────────────────────────────┘               │
│                                                                      │
│  Benefits:                                                           │
│  • Smaller index size (up to 90% reduction)                          │
│  • Fewer page splits                                                 │
│  • Better cache efficiency                                           │
└─────────────────────────────────────────────────────────────────────┘
```

```sql
-- Control deduplication
CREATE INDEX idx_dedup ON logs (level) WITH (deduplicate_items = on);  -- Default

-- Check deduplication status
SELECT * FROM bt_metap('idx_logs_level');
-- allequalimage = t means deduplication possible

-- View posting list stats
SELECT * FROM bt_page_items('idx_logs_level', 1);
```

---

## 4. B-Tree Operators

### Supported Operations

```sql
-- B-Tree supports these operators:
-- <, <=, =, >=, >, BETWEEN

-- Equality
SELECT * FROM users WHERE email = 'john@example.com';

-- Range
SELECT * FROM orders WHERE created_at >= '2024-01-01';
SELECT * FROM products WHERE price BETWEEN 100 AND 500;

-- Pattern matching (prefix only)
SELECT * FROM users WHERE name LIKE 'John%';  -- Uses index
SELECT * FROM users WHERE name LIKE '%John';  -- Cannot use B-tree

-- NULL handling
SELECT * FROM users WHERE email IS NULL;      -- Can use index
SELECT * FROM users WHERE email IS NOT NULL;  -- Can use index

-- NOT EQUAL (usually not efficient)
SELECT * FROM users WHERE status != 'inactive';  -- Often full scan
```

### Operator Classes

```sql
-- Default operator class
CREATE INDEX idx_name ON users (name);  -- Uses text_ops

-- Specify operator class
CREATE INDEX idx_name_pattern ON users (name text_pattern_ops);
-- Required for locale-aware LIKE with non-C locale

-- Common operator classes:
-- text_ops: Standard text comparison
-- text_pattern_ops: LIKE with non-C locale
-- varchar_pattern_ops: Same for varchar
-- bpchar_pattern_ops: Same for char(n)

-- Check operator class
SELECT opcname, opcdefault FROM pg_opclass WHERE opcmethod = (
    SELECT oid FROM pg_am WHERE amname = 'btree'
);
```

---

## 5. Multi-Column Indexes

### Column Order Matters

```sql
-- Multi-column B-Tree
CREATE INDEX idx_orders_user_date ON orders (user_id, created_at);

-- Index structure (conceptual):
-- ┌────────────────────────────────────┐
-- │ (user_1, 2024-01-01)               │
-- │ (user_1, 2024-01-02)               │
-- │ (user_1, 2024-01-03)               │
-- │ (user_2, 2024-01-01)               │
-- │ (user_2, 2024-01-02)               │
-- │ ...                                │
-- └────────────────────────────────────┘

-- Queries that can use this index:
SELECT * FROM orders WHERE user_id = 1;                           -- ✓
SELECT * FROM orders WHERE user_id = 1 AND created_at > '2024-01-01'; -- ✓
SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at;       -- ✓

-- Cannot use efficiently:
SELECT * FROM orders WHERE created_at > '2024-01-01';             -- ✗ (scans all)
```

### Descending Indexes

```sql
-- Mixed sort directions
CREATE INDEX idx_orders_user_date_desc ON orders (user_id ASC, created_at DESC);

-- Optimizes:
SELECT * FROM orders WHERE user_id = 1 ORDER BY created_at DESC;

-- Note: Simple indexes work both directions
-- Multi-column need explicit direction for optimization
```

---

## 6. B-Tree Maintenance

### Index Bloat

```sql
-- Check index bloat
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE indexrelname = 'idx_users_email';

-- Estimate bloat with pgstattuple
CREATE EXTENSION pgstattuple;

SELECT * FROM pgstatindex('idx_users_email');
-- avg_leaf_density: Should be > 90%
-- leaf_fragmentation: Should be low

-- Reindex to remove bloat
REINDEX INDEX idx_users_email;
REINDEX TABLE users;
REINDEX DATABASE mydb;

-- Concurrent reindex (no locks)
REINDEX INDEX CONCURRENTLY idx_users_email;
```

### Dead Tuples and HOT

```sql
-- HOT (Heap-Only Tuples) optimization
-- Updates that don't change indexed columns avoid index updates

-- Check HOT update ratio
SELECT
    relname,
    n_tup_upd,
    n_tup_hot_upd,
    ROUND(100.0 * n_tup_hot_upd / NULLIF(n_tup_upd, 0), 2) AS hot_ratio
FROM pg_stat_user_tables
WHERE relname = 'users';

-- High HOT ratio = fewer index updates = less bloat
```

---

## 7. Performance Considerations

### Index Size Estimation

```sql
-- Estimate index size
SELECT
    pg_size_pretty(
        pg_relation_size('idx_users_email')
    ) AS index_size,
    pg_size_pretty(
        pg_relation_size('users')
    ) AS table_size;

-- Detailed index statistics
SELECT
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size,
    idx_scan AS scans,
    idx_tup_read AS tuples_read
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Fill Factor

```sql
-- Control page fullness
CREATE INDEX idx_users_email ON users (email) WITH (fillfactor = 90);

-- Default is 90% for B-Tree
-- Lower fillfactor:
--   - Leaves room for updates
--   - Reduces page splits
--   - Larger index size
-- Higher fillfactor:
--   - Smaller index
--   - More page splits on updates
```

### Parallel Index Build

```sql
-- Parallel index creation (PostgreSQL 11+)
SET max_parallel_maintenance_workers = 4;

CREATE INDEX idx_large_table ON large_table (column);
-- Uses multiple workers for sorting

-- Check parallel workers used
EXPLAIN (ANALYZE) CREATE INDEX ...;
```

---

## 8. Practical Examples

### Optimizing Equality + Range

```sql
-- Common pattern: Filter by equality, then range
SELECT * FROM orders
WHERE customer_id = 100
  AND order_date BETWEEN '2024-01-01' AND '2024-03-31';

-- Best index:
CREATE INDEX idx_orders_cust_date ON orders (customer_id, order_date);

-- Equality column first (narrows down)
-- Range column second (scan within customer)
```

### Optimizing ORDER BY

```sql
-- Query with ORDER BY
SELECT * FROM products
WHERE category = 'electronics'
ORDER BY price DESC
LIMIT 10;

-- Index that optimizes both filter and sort:
CREATE INDEX idx_products_cat_price ON products (category, price DESC);

-- Query plan shows:
-- Index Scan Backward (no explicit sort needed)
```

### Covering Frequently Used Queries

```sql
-- Frequent query
SELECT id, email, name FROM users WHERE email = ?;

-- Covering index (avoids heap access)
CREATE INDEX idx_users_email_covering ON users (email) INCLUDE (id, name);

-- Query plan shows:
-- Index Only Scan
```

---

## Summary

| Aspect | Details |
|--------|---------|
| Structure | Balanced tree with sorted keys |
| Operations | <, <=, =, >=, >, BETWEEN, LIKE 'prefix%' |
| Complexity | O(log n) search, O(n) range scan |
| Best For | Equality and range queries, sorting |
| Duplicates | Deduplication (v13+) with posting lists |
| Maintenance | REINDEX for bloat, monitor with pg_stat |

---

## Further Reading

- PostgreSQL B-Tree Implementation
- pageinspect extension documentation
- "PostgreSQL 14 Internals" - B-Tree chapter
