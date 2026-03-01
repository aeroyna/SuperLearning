# Index-Only Scans and Covering Indexes

## Learning Objectives
- Understand index-only scans and their benefits
- Create covering indexes with INCLUDE clause
- Optimize queries to avoid heap access
- Monitor and maintain visibility map

---

## 1. Index-Only Scans

### The Problem: Heap Access

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Regular Index Scan                                │
│                                                                      │
│  Query: SELECT id, email FROM users WHERE email = 'john@example.com' │
│                                                                      │
│  Step 1: Search B-Tree index on email                                │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  Index: email → TID (table row pointer)                 │        │
│  │  Found: 'john@example.com' → (block 42, offset 3)       │        │
│  └─────────────────────────────────────────────────────────┘        │
│                    │                                                 │
│                    ▼                                                 │
│  Step 2: Fetch row from heap (table)                                │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  Heap: Read block 42, get row data                      │        │
│  │  Return: id=5, email='john@example.com'                 │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
│  Even though we only need id and email, we had to:                  │
│  1. Read index page(s)                                               │
│  2. Read heap page (extra I/O!)                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### The Solution: Index-Only Scan

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Index-Only Scan                                   │
│                                                                      │
│  Query: SELECT email FROM users WHERE email = 'john@example.com'    │
│                                                                      │
│  Index contains all needed data:                                     │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  Index: email column stored in B-Tree                   │        │
│  │  Found: 'john@example.com'                              │        │
│  │  Return directly from index! (No heap access)           │        │
│  └─────────────────────────────────────────────────────────┘        │
│                                                                      │
│  Benefits:                                                           │
│  • Fewer I/O operations                                              │
│  • Better cache utilization (index pages smaller)                    │
│  • Significant speedup for covered queries                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Requirements for Index-Only Scan

```sql
-- Index-only scan happens when:
-- 1. All SELECT columns are in the index
-- 2. All WHERE columns are in the index
-- 3. Visibility map confirms all-visible pages

-- Example that CAN use index-only scan:
CREATE INDEX idx_users_email ON users (email);

-- Index-only scan possible:
SELECT email FROM users WHERE email LIKE 'john%';
EXPLAIN SELECT email FROM users WHERE email LIKE 'john%';
-- Index Only Scan using idx_users_email

-- NOT index-only (needs 'name' from heap):
SELECT email, name FROM users WHERE email LIKE 'john%';
EXPLAIN SELECT email, name FROM users WHERE email LIKE 'john%';
-- Index Scan using idx_users_email
```

---

## 2. Visibility Map

### What is the Visibility Map?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Visibility Map (VM)                               │
│                                                                      │
│  Each table has a visibility map file (tablename_vm)                │
│  One bit per heap page indicating if ALL tuples are visible         │
│                                                                      │
│  Heap Pages:     [Page 0] [Page 1] [Page 2] [Page 3] [Page 4]       │
│  Visibility Map:    1        1        0        1        1           │
│                   (all     (all     (has     (all     (all          │
│                  visible) visible) updates) visible) visible)       │
│                                                                      │
│  For index-only scan on Page 2:                                      │
│  • VM bit = 0 → Must check heap for visibility                      │
│  • Some tuples might be deleted/updated but not yet vacuumed        │
│                                                                      │
│  For Pages 0, 1, 3, 4:                                               │
│  • VM bit = 1 → Safe to return from index only                      │
│  • No heap access needed                                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Visibility Map and VACUUM

```sql
-- VACUUM updates the visibility map
VACUUM users;

-- Check heap visibility
SELECT
    c.relname,
    pg_size_pretty(pg_relation_size(c.oid)) AS table_size,
    pg_size_pretty(pg_relation_size(c.reltoastrelid)) AS toast_size,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables s
JOIN pg_class c ON s.relid = c.oid
WHERE s.relname = 'users';

-- High n_dead_tup = visibility map outdated
-- Run VACUUM to update VM and enable index-only scans

-- Monitor index-only scan effectiveness
SELECT
    indexrelname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch  -- Lower is better (fewer heap fetches)
FROM pg_stat_user_indexes
WHERE relname = 'users';
```

---

## 3. Covering Indexes (INCLUDE Clause)

### The INCLUDE Clause

```sql
-- PostgreSQL 11+ INCLUDE syntax
CREATE INDEX idx_orders_customer ON orders (customer_id)
INCLUDE (order_date, total, status);

-- Structure:
-- ┌────────────────────────────────────────────────────────────┐
-- │  Key Columns: customer_id (used for searching/ordering)   │
-- │  Included Columns: order_date, total, status (just stored)│
-- └────────────────────────────────────────────────────────────┘

-- Index-only scan possible for:
SELECT customer_id, order_date, total, status
FROM orders
WHERE customer_id = 100;

-- Key difference from multi-column index:
-- INCLUDE columns are NOT used for searching or ordering
-- They're only stored to avoid heap access
```

### INCLUDE vs Multi-Column Index

```sql
-- Multi-column index
CREATE INDEX idx_multi ON orders (customer_id, order_date, total);

-- INCLUDE index
CREATE INDEX idx_include ON orders (customer_id) INCLUDE (order_date, total);

-- Differences:

-- 1. SEARCHABILITY
-- Multi-column: Can search on customer_id, (customer_id, order_date), or all three
-- INCLUDE: Can only search on customer_id

-- 2. ORDERING
-- Multi-column: ORDER BY customer_id, order_date, total (uses index)
-- INCLUDE: ORDER BY customer_id (only, not included columns)

-- 3. SIZE
-- Multi-column: Key includes all columns (larger index entries)
-- INCLUDE: Smaller key, additional columns stored separately

-- 4. UNIQUENESS
-- Multi-column: UNIQUE applies to all columns
CREATE UNIQUE INDEX idx_unique ON orders (customer_id, order_date);
-- INCLUDE: UNIQUE applies only to key columns
CREATE UNIQUE INDEX idx_unique ON orders (customer_id) INCLUDE (order_date);
-- Only customer_id must be unique
```

### When to Use INCLUDE

```sql
-- Good: Query needs extra columns but doesn't search/sort by them
-- Query pattern:
SELECT id, customer_id, total, status
FROM orders
WHERE customer_id = 100;

-- Best index:
CREATE INDEX idx_orders_cust ON orders (customer_id)
INCLUDE (id, total, status);

-- Not good: If you also search or sort by those columns
-- Query pattern:
SELECT * FROM orders WHERE customer_id = 100 ORDER BY total DESC;

-- Better as multi-column:
CREATE INDEX idx_orders_cust_total ON orders (customer_id, total DESC);
```

---

## 4. Practical Examples

### Common Covering Index Patterns

```sql
-- E-commerce: Order lookup
CREATE INDEX idx_orders_user ON orders (user_id)
INCLUDE (order_date, total, status);

-- Query (index-only scan):
SELECT user_id, order_date, total, status
FROM orders
WHERE user_id = 12345;

-- Blog: Post listing
CREATE INDEX idx_posts_author ON posts (author_id, published_at DESC)
INCLUDE (title, excerpt);

-- Query (index-only scan + sorted):
SELECT author_id, published_at, title, excerpt
FROM posts
WHERE author_id = 100
ORDER BY published_at DESC
LIMIT 10;

-- User lookup
CREATE UNIQUE INDEX idx_users_email ON users (email)
INCLUDE (id, name, status);

-- Query (index-only scan):
SELECT id, email, name, status
FROM users
WHERE email = 'john@example.com';
```

### Optimizing Existing Queries

```sql
-- Step 1: Identify frequently executed queries
SELECT
    substring(query, 1, 100) AS query,
    calls,
    mean_exec_time AS avg_time_ms
FROM pg_stat_statements
WHERE query LIKE 'SELECT%FROM orders%'
ORDER BY calls DESC
LIMIT 10;

-- Step 2: Check if index-only scan is possible
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, customer_id, status FROM orders WHERE customer_id = 100;
-- Look for: "Index Scan" vs "Index Only Scan"
-- Check: "Heap Fetches" count

-- Step 3: Create covering index if beneficial
CREATE INDEX idx_orders_cust_covering ON orders (customer_id)
INCLUDE (id, status);

-- Step 4: Verify improvement
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, customer_id, status FROM orders WHERE customer_id = 100;
-- Should now show: "Index Only Scan"
-- Heap Fetches should be 0 (or very low)
```

---

## 5. Monitoring Index-Only Scans

### Checking Heap Fetches

```sql
-- View index scan statistics
SELECT
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS heap_fetches
FROM pg_stat_user_indexes
WHERE idx_scan > 0
ORDER BY idx_tup_fetch DESC;

-- Ratio of heap fetches to tuples read
SELECT
    indexrelname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    CASE
        WHEN idx_tup_read > 0
        THEN ROUND(100.0 * idx_tup_fetch / idx_tup_read, 2)
        ELSE 0
    END AS heap_fetch_ratio
FROM pg_stat_user_indexes
WHERE idx_scan > 100
ORDER BY heap_fetch_ratio DESC;

-- High heap_fetch_ratio indicates:
-- - Need covering index, OR
-- - Need to VACUUM to update visibility map
```

### EXPLAIN Output

```sql
-- Check execution plan
EXPLAIN (ANALYZE, BUFFERS) SELECT id, email FROM users WHERE email = 'test@example.com';

-- Index Only Scan (good):
-- Index Only Scan using idx_users_email on users
--   Index Cond: (email = 'test@example.com')
--   Heap Fetches: 0  ← Key metric!
--   Buffers: shared hit=3

-- Index Scan (needs heap):
-- Index Scan using idx_users_email on users
--   Index Cond: (email = 'test@example.com')
--   Buffers: shared hit=5  ← More buffer hits

-- Heap Fetches > 0 means visibility map indicated some pages not all-visible
-- Run VACUUM to improve
```

### Visibility Map Health

```sql
-- Check table bloat and dead tuples
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;

-- High dead_pct = outdated visibility map
-- VACUUM to clean up
VACUUM ANALYZE tablename;
```

---

## 6. Advanced Techniques

### Covering Index with Expressions

```sql
-- Include computed value
CREATE INDEX idx_orders_customer ON orders (customer_id)
INCLUDE ((quantity * unit_price));

-- Note: Expression must be in parentheses in INCLUDE

-- Query can use index-only scan:
SELECT customer_id, quantity * unit_price AS total
FROM orders
WHERE customer_id = 100;
```

### Partial Covering Index

```sql
-- Combine partial and covering
CREATE INDEX idx_orders_pending ON orders (customer_id)
INCLUDE (order_date, total)
WHERE status = 'pending';

-- Index-only scan for:
SELECT customer_id, order_date, total
FROM orders
WHERE customer_id = 100 AND status = 'pending';
```

### Large Included Columns

```sql
-- Be careful with large columns in INCLUDE
-- This makes the index large:
CREATE INDEX idx_posts_bad ON posts (author_id)
INCLUDE (content);  -- content is TEXT, possibly large!

-- Better: Only include small, frequently needed columns
CREATE INDEX idx_posts_good ON posts (author_id)
INCLUDE (title, published_at, word_count);

-- For large columns, accept heap access
-- Or consider denormalization in specific cases
```

---

## 7. Trade-offs and Considerations

### Benefits vs Costs

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Covering Index Trade-offs                         │
│                                                                      │
│  BENEFITS:                                                           │
│  ✓ Eliminates heap access for covered queries                        │
│  ✓ Reduces I/O significantly                                         │
│  ✓ Better cache efficiency                                           │
│  ✓ Lower query latency                                               │
│                                                                      │
│  COSTS:                                                              │
│  ✗ Larger index size                                                 │
│  ✗ Slower index updates (more columns to maintain)                   │
│  ✗ More memory for caching                                           │
│  ✗ Longer REINDEX time                                               │
│                                                                      │
│  DECISION FACTORS:                                                   │
│  • Query frequency (high frequency = more benefit)                   │
│  • Read vs write ratio (read-heavy = more benefit)                   │
│  • Column sizes (small = more benefit)                               │
│  • Table update frequency (low update = more benefit)                │
└─────────────────────────────────────────────────────────────────────┘
```

### Index Size Comparison

```sql
-- Compare index sizes
-- Simple index
CREATE INDEX idx_simple ON orders (customer_id);

-- Covering index
CREATE INDEX idx_covering ON orders (customer_id)
INCLUDE (order_date, total, status);

-- Check sizes
SELECT
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS size
FROM pg_indexes
WHERE tablename = 'orders'
ORDER BY pg_relation_size(indexname::regclass) DESC;

-- Covering index will be larger
-- Worth it if query performance improvement is significant
```

---

## Summary

| Concept | Description |
|---------|-------------|
| Index-Only Scan | Returns data from index without heap access |
| Visibility Map | Tracks which pages have all-visible tuples |
| INCLUDE Clause | Adds non-key columns to index for coverage |
| Heap Fetches | Metric showing how often heap must be accessed |
| VACUUM | Updates visibility map, enables index-only scans |

---

## Quick Reference

```sql
-- Create covering index
CREATE INDEX idx_name ON table (key_columns) INCLUDE (extra_columns);

-- Check for index-only scan
EXPLAIN (ANALYZE, BUFFERS) SELECT ... ;
-- Look for: Index Only Scan, Heap Fetches: 0

-- Monitor effectiveness
SELECT indexrelname, idx_tup_fetch FROM pg_stat_user_indexes;

-- Maintain visibility map
VACUUM ANALYZE table_name;
```

---

## Further Reading

- PostgreSQL Index-Only Scans documentation
- INCLUDE clause documentation
- Visibility Map internals
