# EXPLAIN and Query Plans

## Learning Objectives
- Master EXPLAIN output interpretation
- Understand query execution plans
- Learn EXPLAIN ANALYZE for actual execution metrics
- Use EXPLAIN to guide optimization decisions

---

## 1. EXPLAIN Basics

### Basic Syntax

```sql
-- Basic EXPLAIN
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Extended format (deprecated, use FORMAT options)
EXPLAIN EXTENDED SELECT * FROM users WHERE email = 'test@example.com';

-- JSON format (more details)
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE email = 'test@example.com';

-- Tree format (MySQL 8.0.16+)
EXPLAIN FORMAT=TREE SELECT * FROM users WHERE email = 'test@example.com';

-- EXPLAIN ANALYZE (MySQL 8.0.18+) - Actually executes query
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';
```

---

## 2. EXPLAIN Output Columns

### Column Reference

```sql
EXPLAIN SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending';

+----+-------------+-------+--------+---------------+---------+---------+-----------+------+-------------+
| id | select_type | table | type   | possible_keys | key     | key_len | ref       | rows | Extra       |
+----+-------------+-------+--------+---------------+---------+---------+-----------+------+-------------+
|  1 | SIMPLE      | o     | ref    | idx_status    | idx_status | 1     | const     | 150  | Using where |
|  1 | SIMPLE      | u     | eq_ref | PRIMARY       | PRIMARY | 4       | db.o.user_id | 1 | NULL        |
+----+-------------+-------+--------+---------------+---------+---------+-----------+------+-------------+
```

### id Column

```sql
-- Same id = executed together (JOIN)
-- Different id = subquery/derived table
-- Higher id = executed first

-- Example with subquery:
EXPLAIN SELECT * FROM orders
WHERE user_id IN (SELECT id FROM users WHERE status = 'active');

+----+-------------+--------+
| id | select_type | table  |
+----+-------------+--------+
|  1 | SIMPLE      | users  |  -- Subquery optimized to semi-join
|  1 | SIMPLE      | orders |
+----+-------------+--------+

-- Or with derived table:
EXPLAIN SELECT * FROM (SELECT user_id, COUNT(*) FROM orders GROUP BY user_id) t;

+----+-------------+------------+
| id | select_type | table      |
+----+-------------+------------+
|  1 | PRIMARY     | <derived2> |  -- Using result of id=2
|  2 | DERIVED     | orders     |  -- Executed first
+----+-------------+------------+
```

### select_type Column

| Value | Description |
|-------|-------------|
| SIMPLE | Simple SELECT (no subqueries or UNION) |
| PRIMARY | Outermost SELECT |
| SUBQUERY | First SELECT in subquery |
| DERIVED | Derived table (FROM subquery) |
| UNION | Second or later SELECT in UNION |
| UNION RESULT | Result of UNION |
| DEPENDENT SUBQUERY | Subquery dependent on outer query |
| MATERIALIZED | Materialized subquery |

### type Column (Access Method)

From best to worst:

```
┌────────────────────────────────────────────────────────────────────┐
│  Access Type Performance (Best to Worst)                           │
│                                                                     │
│  system    → Table has exactly one row (constant)                   │
│  const     → At most one row (PK/UNIQUE lookup)                     │
│  eq_ref    → One row per row from previous table (JOIN on PK)       │
│  ref       → Multiple rows possible (non-unique index)              │
│  fulltext  → Full-text index                                        │
│  ref_or_null → Like ref, plus NULL values                           │
│  index_merge → Multiple indexes merged                              │
│  range     → Index range scan (BETWEEN, IN, >, <)                   │
│  index     → Full index scan (all rows via index)                   │
│  ALL       → Full table scan (worst!)                               │
│                                                                     │
│  Goal: aim for const, eq_ref, ref, or range                         │
└────────────────────────────────────────────────────────────────────┘
```

```sql
-- const: Primary key lookup
EXPLAIN SELECT * FROM users WHERE id = 1;
-- type: const

-- eq_ref: JOIN on primary key
EXPLAIN SELECT * FROM orders o JOIN users u ON o.user_id = u.id;
-- type: eq_ref for users table

-- ref: Non-unique index
EXPLAIN SELECT * FROM orders WHERE status = 'pending';
-- type: ref (assuming idx_status exists)

-- range: Index range scan
EXPLAIN SELECT * FROM orders WHERE created_at > '2024-01-01';
-- type: range

-- ALL: Full table scan (BAD!)
EXPLAIN SELECT * FROM orders WHERE description LIKE '%keyword%';
-- type: ALL
```

### key and key_len Columns

```sql
-- key: Index actually used
-- key_len: Bytes of index used (helps identify partial index usage)

-- Full composite index usage:
CREATE INDEX idx_a_b_c ON table (a, b, c);
EXPLAIN SELECT * FROM table WHERE a = 1 AND b = 2 AND c = 3;
-- key_len shows all 3 columns used

-- Partial usage:
EXPLAIN SELECT * FROM table WHERE a = 1 AND b = 2;
-- key_len shows only a and b used

-- Key length calculation:
-- INT: 4 bytes
-- BIGINT: 8 bytes
-- CHAR(n): n × charset_bytes (utf8mb4 = 4)
-- VARCHAR(n): n × charset_bytes + 2 (length prefix)
-- NULL column: +1 byte
```

### rows Column

```sql
-- Estimated rows to examine (can be inaccurate!)
-- Lower is better

EXPLAIN SELECT * FROM orders WHERE status = 'pending';
-- rows: 1500 (estimated)

-- Compare with actual:
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';
-- Shows: (actual rows=1432)
```

### Extra Column

Important values:

| Value | Meaning |
|-------|---------|
| Using index | Covering index (excellent!) |
| Using where | Filter applied after reading |
| Using temporary | Temp table needed |
| Using filesort | Extra sorting pass needed |
| Using index condition | Index Condition Pushdown |
| Using join buffer | Block nested loop join |
| Impossible WHERE | Query returns no rows |

```sql
-- Good: Using index (covering index)
EXPLAIN SELECT email FROM users WHERE email = 'test@example.com';
-- Extra: Using index

-- Needs attention: Using filesort
EXPLAIN SELECT * FROM orders ORDER BY total;
-- Extra: Using filesort (add index on 'total')

-- Needs attention: Using temporary
EXPLAIN SELECT DISTINCT status FROM orders;
-- Extra: Using temporary; Using filesort
```

---

## 3. EXPLAIN ANALYZE

### Actual Execution Metrics (MySQL 8.0.18+)

```sql
EXPLAIN ANALYZE SELECT * FROM orders
WHERE status = 'pending'
ORDER BY created_at DESC
LIMIT 10;

-> Limit: 10 row(s)  (cost=150.00 rows=10) (actual time=0.200..0.500 rows=10 loops=1)
    -> Sort: orders.created_at DESC  (cost=150.00 rows=1000) (actual time=0.195..0.450 rows=10 loops=1)
        -> Filter: (orders.status = 'pending')  (cost=100.00 rows=1000) (actual time=0.050..0.180 rows=150 loops=1)
            -> Index lookup on orders using idx_status (status='pending')  (cost=100.00 rows=1000) (actual time=0.040..0.150 rows=150 loops=1)

-- Key metrics:
-- cost: Optimizer's cost estimate
-- actual time: First row..last row timing (ms)
-- rows: Estimated vs actual row count
-- loops: Times this operation repeated
```

### Reading EXPLAIN ANALYZE Output

```sql
-- Read from innermost to outermost (bottom-up in tree format)

EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as order_count
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
GROUP BY u.id
HAVING order_count > 5;

-- Output shows:
-- 1. Index lookup on orders (first operation)
-- 2. Join with users
-- 3. Aggregate (GROUP BY)
-- 4. Filter (HAVING)
-- 5. Return results

-- Look for:
-- - Large "rows" differences between estimated and actual
-- - High "actual time" values
-- - High "loops" counts (executed many times)
```

---

## 4. JSON Format Details

```sql
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE status = 'pending';

{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "150.00"
    },
    "table": {
      "table_name": "orders",
      "access_type": "ref",
      "possible_keys": ["idx_status"],
      "key": "idx_status",
      "key_length": "1",
      "ref": ["const"],
      "rows_examined_per_scan": 1000,
      "rows_produced_per_join": 1000,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "50.00",
        "eval_cost": "100.00",
        "prefix_cost": "150.00",
        "data_read_per_join": "1M"
      },
      "used_columns": ["id", "user_id", "status", "total", "created_at"]
    }
  }
}
```

### Key JSON Properties

```sql
-- cost_info: Optimizer cost breakdown
-- filtered: Percentage of rows passing condition
-- used_columns: Columns actually read
-- attached_condition: WHERE clause applied
```

---

## 5. Common Execution Plan Patterns

### Full Table Scan (Avoid)

```sql
EXPLAIN SELECT * FROM orders WHERE YEAR(created_at) = 2024;

+----+------+------+------+---------+
| id | type | key  | rows | Extra   |
+----+------+------+------+---------+
|  1 | ALL  | NULL | 100000 | Using where |
+----+------+------+------+---------+

-- Problem: Function on column prevents index use
-- Solution: Use range condition
SELECT * FROM orders
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

### Filesort (Sometimes Unavoidable)

```sql
EXPLAIN SELECT * FROM orders WHERE status = 'pending' ORDER BY total;

+----+------+-----------+------+------+-----------------------------+
| id | type | key       | rows | Extra                             |
+----+------+-----------+------+------+-----------------------------+
|  1 | ref  | idx_status| 1000 | Using where; Using filesort       |
+----+------+-----------+------+------+-----------------------------+

-- Problem: Index on status, but sorting by total
-- Solution: Composite index
CREATE INDEX idx_status_total ON orders (status, total);
```

### Temporary Table (Expensive)

```sql
EXPLAIN SELECT user_id, COUNT(*) FROM orders
GROUP BY user_id
ORDER BY COUNT(*) DESC;

+----+------+-------+------+------+-------------------------------+
| id | type | key   | rows | Extra                               |
+----+------+-------+------+------+-------------------------------+
|  1 | ALL  | NULL  | 100000 | Using temporary; Using filesort |
+----+------+-------+------+------+-------------------------------+

-- Solution: Add index for GROUP BY
CREATE INDEX idx_user ON orders (user_id);
```

### Covering Index (Excellent)

```sql
EXPLAIN SELECT user_id, status FROM orders WHERE status = 'pending';

+----+------+-------------------+------+-------------+
| id | type | key               | rows | Extra       |
+----+------+-------------------+------+-------------+
|  1 | ref  | idx_status_user_id| 1000 | Using index |
+----+------+-------------------+------+-------------+

-- "Using index" = No table access needed!
-- All data comes from index
```

---

## 6. Optimizer Hints

### Index Hints

```sql
-- Force index usage
EXPLAIN SELECT * FROM orders FORCE INDEX (idx_created)
WHERE status = 'pending' AND created_at > '2024-01-01';

-- Ignore index
EXPLAIN SELECT * FROM orders IGNORE INDEX (idx_status)
WHERE status = 'pending';

-- Use index (suggestion)
EXPLAIN SELECT * FROM orders USE INDEX (idx_status)
WHERE status = 'pending';
```

### MySQL 8.0 Optimizer Hints

```sql
-- Join order hint
EXPLAIN SELECT /*+ JOIN_ORDER(users, orders) */ *
FROM users u JOIN orders o ON u.id = o.user_id;

-- Index hint
EXPLAIN SELECT /*+ INDEX(orders idx_status) */ *
FROM orders WHERE status = 'pending';

-- No index hint
EXPLAIN SELECT /*+ NO_INDEX(orders idx_status) */ *
FROM orders WHERE status = 'pending';

-- Disable specific optimizations
EXPLAIN SELECT /*+ NO_MERGE(derived) */ *
FROM (SELECT * FROM orders WHERE status = 'pending') derived;
```

---

## 7. Visual EXPLAIN Tools

### MySQL Workbench

```sql
-- Execute query with Visual Explain
-- Shows graphical query plan with color coding:
-- Green: Good performance
-- Yellow: Potential issues
-- Red: Performance problems
```

### sys Schema

```sql
-- Query analysis using sys schema
SELECT * FROM sys.statement_analysis LIMIT 10;

-- Statements with full table scans
SELECT * FROM sys.statements_with_full_table_scans LIMIT 10;

-- Statements with temp tables
SELECT * FROM sys.statements_with_temp_tables LIMIT 10;
```

---

## 8. Practical Examples

### Debugging a Slow Query

```sql
-- Original slow query
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as orders, SUM(o.total) as revenue
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= '2024-01-01'
GROUP BY u.id
ORDER BY revenue DESC
LIMIT 100;

-- Analyze the plan:
-- 1. Check access type (ALL is bad)
-- 2. Check rows estimates
-- 3. Look for filesort/temporary
-- 4. Check if indexes are used

-- After optimization (add index and rewrite):
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at, total);

EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) as orders, SUM(o.total) as revenue
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= '2024-01-01'
GROUP BY u.id
ORDER BY revenue DESC
LIMIT 100;
```

---

## Summary

| EXPLAIN Column | What to Check |
|----------------|---------------|
| type | Should be const/ref/range, not ALL |
| key | Should show an index being used |
| rows | Lower is better |
| Extra | "Using index" good, "Using filesort/temporary" concerning |

### Key Takeaways

1. **Use EXPLAIN ANALYZE** for actual execution metrics
2. **Avoid type=ALL** (full table scans)
3. **Look for "Using index"** in Extra column
4. **Monitor rows** column for optimizer accuracy
5. **Use JSON format** for detailed cost breakdown

---

## Further Reading

- MySQL EXPLAIN documentation
- "High Performance MySQL" - Query optimization
- MySQL Optimizer Trace (for deep debugging)
