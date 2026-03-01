# Index Optimization in MySQL

## Learning Objectives
- Master index design best practices
- Learn to identify and fix index problems
- Understand index maintenance strategies
- Optimize indexes for specific query patterns

---

## 1. Index Design Principles

### The ESR Rule: Equality, Sort, Range

```sql
-- Order columns by access pattern:
-- 1. Equality conditions (WHERE col = value)
-- 2. Sort columns (ORDER BY)
-- 3. Range conditions (WHERE col > value)

-- Example query:
SELECT * FROM orders
WHERE status = 'pending'           -- Equality
  AND created_at > '2024-01-01'    -- Range
ORDER BY priority DESC;            -- Sort

-- Optimal index following ESR:
CREATE INDEX idx_status_priority_created
ON orders (status, priority DESC, created_at);

-- Why this order works:
-- status = 'pending': Direct lookup
-- priority DESC: Already sorted (no filesort)
-- created_at > date: Range scan on remaining rows
```

### Don't Repeat the Primary Key

```sql
-- InnoDB automatically includes PK in secondary indexes
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    name VARCHAR(100)
);

-- Wrong: Redundant 'id'
CREATE INDEX idx_email_id ON users (email, id);  -- ✗ Wasteful

-- Right: 'id' is automatically included
CREATE INDEX idx_email ON users (email);  -- ✓

-- idx_email actually stores (email, id) internally
```

### Avoid Redundant Indexes

```sql
-- Identify redundant indexes
SELECT
    t.TABLE_SCHEMA,
    t.TABLE_NAME,
    s.INDEX_NAME AS redundant_index,
    s2.INDEX_NAME AS covering_index
FROM information_schema.STATISTICS s
JOIN information_schema.STATISTICS s2
    ON s.TABLE_SCHEMA = s2.TABLE_SCHEMA
    AND s.TABLE_NAME = s2.TABLE_NAME
    AND s.COLUMN_NAME = s2.COLUMN_NAME
    AND s.SEQ_IN_INDEX = s2.SEQ_IN_INDEX
    AND s.INDEX_NAME != s2.INDEX_NAME
JOIN information_schema.TABLES t
    ON s.TABLE_SCHEMA = t.TABLE_SCHEMA
    AND s.TABLE_NAME = t.TABLE_NAME
WHERE s.SEQ_IN_INDEX = 1
    AND s.TABLE_SCHEMA = 'mydb';

-- Examples of redundant indexes:
-- INDEX (a) is redundant if INDEX (a, b) exists
-- INDEX (a, b) is redundant if INDEX (a, b, c) exists
```

---

## 2. Analyzing Index Usage

### Finding Unused Indexes

```sql
-- MySQL 8.0: Using sys schema
SELECT * FROM sys.schema_unused_indexes
WHERE object_schema = 'mydb';

-- Using performance_schema
SELECT
    object_schema,
    object_name,
    index_name,
    count_read,
    count_write
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema = 'mydb'
    AND index_name IS NOT NULL
    AND count_read = 0
ORDER BY object_name, index_name;

-- Unused indexes waste:
-- - Storage space
-- - Write performance (must update on INSERT/UPDATE)
-- - Buffer pool memory
```

### Finding Missing Indexes

```sql
-- Check slow query log for queries without index
SET GLOBAL slow_query_log = 1;
SET GLOBAL log_queries_not_using_indexes = 1;
SET GLOBAL long_query_time = 1;

-- Review slow query log
-- /var/log/mysql/slow.log

-- Or use Performance Schema
SELECT
    DIGEST_TEXT,
    COUNT_STAR,
    SUM_NO_INDEX_USED,
    SUM_NO_GOOD_INDEX_USED,
    AVG_TIMER_WAIT/1000000000 as avg_ms
FROM performance_schema.events_statements_summary_by_digest
WHERE SUM_NO_INDEX_USED > 0
   OR SUM_NO_GOOD_INDEX_USED > 0
ORDER BY COUNT_STAR DESC
LIMIT 20;
```

### Index Cardinality Analysis

```sql
-- Check index cardinality
SELECT
    TABLE_NAME,
    INDEX_NAME,
    GROUP_CONCAT(COLUMN_NAME ORDER BY SEQ_IN_INDEX) AS columns,
    MAX(CARDINALITY) AS cardinality
FROM information_schema.STATISTICS
WHERE TABLE_SCHEMA = 'mydb'
GROUP BY TABLE_NAME, INDEX_NAME
ORDER BY TABLE_NAME, cardinality;

-- Low cardinality indexes may not be useful
-- Exception: Combined with other conditions or for covering
```

---

## 3. Query Pattern Optimization

### Optimizing Equality Queries

```sql
-- Simple equality - single column index
SELECT * FROM users WHERE email = 'test@example.com';
CREATE INDEX idx_email ON users (email);

-- Multiple equalities - composite index
SELECT * FROM orders WHERE user_id = 123 AND status = 'pending';
CREATE INDEX idx_user_status ON orders (user_id, status);
-- Column order: Most selective first? Not always...
-- Column order: Most frequently queried alone first (leftmost prefix rule)
```

### Optimizing Range Queries

```sql
-- Range query stops index usage for subsequent columns
SELECT * FROM logs
WHERE created_at > '2024-01-01'
  AND level = 'ERROR';

-- This index won't use 'level' after range on created_at:
CREATE INDEX idx_date_level ON logs (created_at, level);  -- ✗

-- Better: Put range column last
CREATE INDEX idx_level_date ON logs (level, created_at);  -- ✓

-- Even better if level has few values:
-- Query all ERROR logs, then filter by date (still uses range)
```

### Optimizing ORDER BY

```sql
-- Index can eliminate filesort
SELECT * FROM products
WHERE category_id = 5
ORDER BY price ASC;

-- Index for both filter and sort:
CREATE INDEX idx_category_price ON products (category_id, price);

-- Descending order matters (MySQL 8.0+):
SELECT * FROM products
ORDER BY created_at DESC, name ASC;

CREATE INDEX idx_created_name ON products (created_at DESC, name ASC);

-- Mixed directions require matching index or filesort
```

### Optimizing GROUP BY

```sql
-- GROUP BY can use loose index scan
SELECT category_id, COUNT(*)
FROM products
GROUP BY category_id;

-- Index enables "Using index for group-by":
CREATE INDEX idx_category ON products (category_id);

-- With conditions:
SELECT category_id, MAX(price)
FROM products
WHERE in_stock = TRUE
GROUP BY category_id;

CREATE INDEX idx_stock_category_price ON products (in_stock, category_id, price);
```

---

## 4. Common Index Anti-Patterns

### Functions on Indexed Columns

```sql
-- Index on column, but function prevents usage
CREATE INDEX idx_date ON orders (created_at);

-- Bad: Function on column
SELECT * FROM orders WHERE YEAR(created_at) = 2024;  -- Full scan

-- Good: Range condition
SELECT * FROM orders
WHERE created_at >= '2024-01-01'
  AND created_at < '2025-01-01';  -- Uses index

-- MySQL 8.0: Functional indexes
CREATE INDEX idx_year ON orders ((YEAR(created_at)));
SELECT * FROM orders WHERE YEAR(created_at) = 2024;  -- Uses index
```

### Implicit Type Conversion

```sql
-- Column type mismatch causes implicit conversion
CREATE TABLE users (
    phone VARCHAR(20),
    INDEX idx_phone (phone)
);

-- Bad: Numeric comparison on VARCHAR
SELECT * FROM users WHERE phone = 1234567890;  -- Full scan!
-- MySQL converts all VARCHAR values to numbers for comparison

-- Good: Match types
SELECT * FROM users WHERE phone = '1234567890';  -- Uses index
```

### Collation Mismatch

```sql
-- Different collations prevent index usage
CREATE TABLE t1 (
    name VARCHAR(100) COLLATE utf8mb4_unicode_ci,
    INDEX idx_name (name)
);

CREATE TABLE t2 (
    name VARCHAR(100) COLLATE utf8mb4_general_ci
);

-- Join may not use index due to collation mismatch
SELECT * FROM t1 JOIN t2 ON t1.name = t2.name;

-- Solution: Match collations or use COLLATE in query
SELECT * FROM t1 JOIN t2 ON t1.name = t2.name COLLATE utf8mb4_unicode_ci;
```

### OR Conditions

```sql
-- OR on different columns often can't use indexes efficiently
SELECT * FROM users WHERE email = 'a@b.com' OR phone = '123';

-- Solutions:
-- 1. UNION (uses separate indexes)
SELECT * FROM users WHERE email = 'a@b.com'
UNION
SELECT * FROM users WHERE phone = '123';

-- 2. Index merge (if optimizer chooses)
CREATE INDEX idx_email ON users (email);
CREATE INDEX idx_phone ON users (phone);
-- EXPLAIN may show: Using union(idx_email, idx_phone)
```

---

## 5. Index Maintenance

### Updating Statistics

```sql
-- Update table statistics for optimizer
ANALYZE TABLE users;

-- Check when stats were last updated
SELECT
    TABLE_NAME,
    UPDATE_TIME,
    TABLE_ROWS
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb';

-- InnoDB samples pages for statistics
SHOW VARIABLES LIKE 'innodb_stats_persistent_sample_pages';
SET GLOBAL innodb_stats_persistent_sample_pages = 100;  -- More accurate
```

### Handling Fragmentation

```sql
-- Check fragmentation
SELECT
    TABLE_NAME,
    DATA_LENGTH,
    INDEX_LENGTH,
    DATA_FREE,
    (DATA_FREE / (DATA_LENGTH + INDEX_LENGTH)) * 100 AS frag_percent
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb'
    AND DATA_FREE > 0
ORDER BY frag_percent DESC;

-- Rebuild table (defragment)
ALTER TABLE users ENGINE = InnoDB;

-- Or use OPTIMIZE
OPTIMIZE TABLE users;

-- Online DDL (minimal locking)
ALTER TABLE users ENGINE = InnoDB, ALGORITHM = INPLACE, LOCK = NONE;
```

### Online Index Operations

```sql
-- Add index online (InnoDB)
ALTER TABLE large_table ADD INDEX idx_new (column1),
    ALGORITHM = INPLACE,
    LOCK = NONE;

-- Drop index online
ALTER TABLE large_table DROP INDEX idx_old,
    ALGORITHM = INPLACE,
    LOCK = NONE;

-- Rename index (MySQL 5.7+)
ALTER TABLE users RENAME INDEX idx_old TO idx_new;

-- For large tables, use pt-online-schema-change
-- (Percona Toolkit)
```

---

## 6. Invisible Indexes (MySQL 8.0)

### Testing Index Removal

```sql
-- Make index invisible (optimizer ignores it)
ALTER TABLE users ALTER INDEX idx_status INVISIBLE;

-- Test queries that might use this index
EXPLAIN SELECT * FROM users WHERE status = 'active';
-- Should show different plan or full scan

-- Check performance impact
-- If no degradation, safe to drop

-- Make visible again
ALTER TABLE users ALTER INDEX idx_status VISIBLE;

-- Or drop if not needed
DROP INDEX idx_status ON users;
```

### Testing New Indexes

```sql
-- Create invisible index first
CREATE INDEX idx_new ON users (email, status) INVISIBLE;

-- Test with optimizer hint
SELECT /*+ USE_INDEX(users idx_new) */ *
FROM users WHERE email = 'test@example.com';

-- If beneficial, make visible
ALTER TABLE users ALTER INDEX idx_new VISIBLE;
```

---

## 7. Index Hints

### Forcing Index Usage

```sql
-- Force specific index
SELECT * FROM orders FORCE INDEX (idx_status)
WHERE status = 'pending' AND total > 100;

-- Suggest index (optimizer may still choose differently)
SELECT * FROM orders USE INDEX (idx_status)
WHERE status = 'pending';

-- Ignore specific index
SELECT * FROM orders IGNORE INDEX (idx_status)
WHERE status = 'pending';
```

### MySQL 8.0 Optimizer Hints

```sql
-- Index hints in comment syntax
SELECT /*+ INDEX(orders idx_status) */ * FROM orders WHERE status = 'pending';

-- Join order hints
SELECT /*+ JOIN_ORDER(users, orders) */ *
FROM users, orders
WHERE users.id = orders.user_id;

-- Disable specific optimizations
SELECT /*+ NO_INDEX_MERGE(orders) */ *
FROM orders WHERE status = 'pending' OR total > 100;

-- Set optimizer switch for query
SELECT /*+ SET_VAR(optimizer_switch='index_merge=off') */ *
FROM orders WHERE status = 'pending' OR total > 100;
```

---

## 8. Monitoring and Diagnostics

### Index Usage Monitoring

```sql
-- Enable index usage statistics
UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME = 'global_instrumentation';

-- View index usage
SELECT
    OBJECT_SCHEMA,
    OBJECT_NAME,
    INDEX_NAME,
    COUNT_STAR AS total_accesses,
    COUNT_READ,
    COUNT_WRITE,
    COUNT_FETCH
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE OBJECT_SCHEMA = 'mydb'
ORDER BY COUNT_STAR DESC;
```

### Query Plan Analysis

```sql
-- Basic EXPLAIN
EXPLAIN SELECT * FROM orders WHERE status = 'pending';

-- EXPLAIN ANALYZE (MySQL 8.0.18+) - Actually executes
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';

-- JSON format for detailed info
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE status = 'pending';

-- Visual EXPLAIN in MySQL Workbench
```

### Performance Schema for Index Issues

```sql
-- Queries not using indexes
SELECT
    DIGEST_TEXT,
    COUNT_STAR,
    SUM_TIMER_WAIT/1000000000000 AS total_seconds,
    SUM_NO_INDEX_USED,
    FIRST_SEEN,
    LAST_SEEN
FROM performance_schema.events_statements_summary_by_digest
WHERE SCHEMA_NAME = 'mydb'
    AND SUM_NO_INDEX_USED > 0
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;
```

---

## 9. Index Optimization Checklist

### Before Creating an Index

```markdown
1. [ ] Identify the queries that need optimization
2. [ ] Check if existing index can be extended
3. [ ] Follow ESR rule (Equality, Sort, Range)
4. [ ] Consider covering index benefits
5. [ ] Evaluate write performance impact
6. [ ] Check for redundant indexes
7. [ ] Estimate index size
```

### After Creating an Index

```markdown
1. [ ] Verify with EXPLAIN that index is used
2. [ ] Test query performance improvement
3. [ ] Monitor write performance
4. [ ] Update statistics (ANALYZE TABLE)
5. [ ] Document the index purpose
```

### Regular Maintenance

```markdown
1. [ ] Review unused indexes monthly
2. [ ] Check for fragmentation
3. [ ] Update statistics after bulk operations
4. [ ] Review slow query log
5. [ ] Audit index effectiveness
```

---

## Summary

| Practice | Benefit |
|----------|---------|
| ESR column order | Optimal index utilization |
| Remove unused indexes | Save storage, faster writes |
| Avoid functions on columns | Index can be used |
| Match data types | Prevent implicit conversion |
| Use covering indexes | Eliminate table lookups |
| Invisible indexes | Safe testing before changes |

---

## Further Reading

- "High Performance MySQL" - Indexing best practices
- MySQL Performance Tuning documentation
- Percona Database Performance Blog
- "Use The Index, Luke" - SQL Indexing tutorial
