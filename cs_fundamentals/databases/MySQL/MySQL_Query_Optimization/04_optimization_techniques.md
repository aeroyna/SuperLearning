# Query Optimization Techniques

## Learning Objectives
- Master practical query optimization patterns
- Learn to rewrite inefficient queries
- Understand schema design for performance
- Apply optimization systematically

---

## 1. Index-Based Optimizations

### Creating Effective Indexes

```sql
-- Analyze query patterns first
-- Query 1: WHERE status = 'pending' ORDER BY created_at
-- Query 2: WHERE user_id = 123 AND status = 'pending'
-- Query 3: WHERE created_at > '2024-01-01'

-- Create indexes to cover common patterns
CREATE INDEX idx_status_created ON orders (status, created_at);
CREATE INDEX idx_user_status ON orders (user_id, status);
CREATE INDEX idx_created ON orders (created_at);

-- Verify usage
EXPLAIN SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at;
```

### Composite Index Column Order

```sql
-- Wrong order: Range column first
CREATE INDEX idx_bad ON orders (created_at, status);
-- Query: WHERE status = 'pending' AND created_at > '2024-01-01'
-- Can only use created_at, not status

-- Right order: Equality before range
CREATE INDEX idx_good ON orders (status, created_at);
-- Uses both columns effectively
```

### Covering Indexes

```sql
-- Before: Requires table lookup
EXPLAIN SELECT id, email, status FROM users WHERE email = 'test@example.com';
-- type: ref, Extra: NULL (table access needed)

-- After: All columns in index
CREATE INDEX idx_email_status ON users (email, status, id);
EXPLAIN SELECT id, email, status FROM users WHERE email = 'test@example.com';
-- type: ref, Extra: Using index (index-only scan)
```

---

## 2. Query Rewriting Techniques

### Avoid Functions on Indexed Columns

```sql
-- Bad: Function prevents index usage
SELECT * FROM orders WHERE YEAR(created_at) = 2024;
-- Full table scan!

-- Good: Use range condition
SELECT * FROM orders
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
-- Uses index on created_at
```

### Replace OR with UNION

```sql
-- Bad: OR on different columns
SELECT * FROM users WHERE email = 'a@b.com' OR phone = '123456';
-- May not use indexes efficiently

-- Good: UNION with separate indexes
SELECT * FROM users WHERE email = 'a@b.com'
UNION ALL
SELECT * FROM users WHERE phone = '123456' AND email != 'a@b.com';
-- Uses idx_email and idx_phone separately
```

### Optimize IN Clauses

```sql
-- Bad: Large IN list
SELECT * FROM products WHERE id IN (1, 2, 3, ..., 10000);

-- Good: Join with temp table or derived table
CREATE TEMPORARY TABLE tmp_ids (id INT PRIMARY KEY);
INSERT INTO tmp_ids VALUES (1), (2), (3), ...;
SELECT p.* FROM products p JOIN tmp_ids t ON p.id = t.id;

-- Or use EXISTS for correlated queries
SELECT * FROM products p
WHERE EXISTS (SELECT 1 FROM categories c WHERE c.id = p.category_id AND c.active = 1);
```

### Avoid SELECT *

```sql
-- Bad: Fetches all columns
SELECT * FROM orders WHERE status = 'pending';

-- Good: Only needed columns
SELECT id, user_id, total, created_at FROM orders WHERE status = 'pending';

-- Benefits:
-- 1. Less data transferred
-- 2. Can use covering indexes
-- 3. Clearer intent
```

---

## 3. JOIN Optimization

### Join Order and Type

```sql
-- Let optimizer choose join order (usually best)
SELECT * FROM orders o JOIN users u ON o.user_id = u.id;

-- Force join order if needed
SELECT STRAIGHT_JOIN * FROM orders o JOIN users u ON o.user_id = u.id;

-- Or use optimizer hint (MySQL 8.0)
SELECT /*+ JOIN_ORDER(orders, users) */ * FROM orders o JOIN users u ON o.user_id = u.id;
```

### Optimize Subqueries to JOINs

```sql
-- Bad: Correlated subquery (runs for each row)
SELECT *
FROM orders o
WHERE o.total > (
    SELECT AVG(total)
    FROM orders o2
    WHERE o2.user_id = o.user_id
);

-- Good: Rewrite as JOIN with derived table
SELECT o.*
FROM orders o
JOIN (
    SELECT user_id, AVG(total) as avg_total
    FROM orders
    GROUP BY user_id
) avg ON o.user_id = avg.user_id
WHERE o.total > avg.avg_total;
```

### EXISTS vs IN

```sql
-- EXISTS (usually better for large outer table)
SELECT * FROM orders o
WHERE EXISTS (
    SELECT 1 FROM users u
    WHERE u.id = o.user_id AND u.status = 'active'
);

-- IN (can be better for small subquery result)
SELECT * FROM orders
WHERE user_id IN (
    SELECT id FROM users WHERE status = 'active'
);

-- MySQL optimizer often transforms these automatically
-- Use EXPLAIN to verify
```

---

## 4. Pagination Optimization

### Offset Pagination Problems

```sql
-- Bad: Large OFFSET is slow
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10 OFFSET 100000;
-- Must scan and skip 100,000 rows!

-- Better: Keyset/Cursor pagination
SELECT * FROM orders
WHERE created_at < '2024-01-15 10:30:00'  -- Last seen value
ORDER BY created_at DESC
LIMIT 10;
-- Uses index directly to starting point
```

### Deferred Join Pattern

```sql
-- Bad: Fetches all columns for skipped rows
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10 OFFSET 10000;

-- Good: Fetch IDs first, then join
SELECT o.*
FROM orders o
JOIN (
    SELECT id FROM orders ORDER BY created_at DESC LIMIT 10 OFFSET 10000
) ids ON o.id = ids.id
ORDER BY o.created_at DESC;
-- Inner query uses covering index (id, created_at)
```

### COUNT Optimization

```sql
-- Slow: Counting with conditions
SELECT COUNT(*) FROM orders WHERE status = 'pending';

-- If approximate is okay, use table stats
EXPLAIN SELECT COUNT(*) FROM orders;  -- Shows row estimate

-- Or maintain counter table
CREATE TABLE order_counts (
    status VARCHAR(20) PRIMARY KEY,
    count INT
);
-- Update via triggers or application
```

---

## 5. Aggregate Query Optimization

### Loose Index Scan

```sql
-- Can use loose index scan for MIN/MAX
SELECT category_id, MAX(price) FROM products GROUP BY category_id;
-- With index on (category_id, price)
-- Extra: Using index for group-by

-- Conditions that enable loose index scan:
-- 1. Single table query
-- 2. GROUP BY uses leftmost index prefix
-- 3. Only MIN/MAX aggregate functions
-- 4. Any other columns are constants
```

### Pre-aggregated Tables

```sql
-- For expensive aggregations, pre-compute
CREATE TABLE daily_sales_summary (
    date DATE PRIMARY KEY,
    total_orders INT,
    total_revenue DECIMAL(15,2),
    avg_order_value DECIMAL(10,2)
);

-- Populate via scheduled job or trigger
INSERT INTO daily_sales_summary
SELECT
    DATE(created_at) as date,
    COUNT(*) as total_orders,
    SUM(total) as total_revenue,
    AVG(total) as avg_order_value
FROM orders
WHERE DATE(created_at) = CURDATE() - INTERVAL 1 DAY
ON DUPLICATE KEY UPDATE
    total_orders = VALUES(total_orders),
    total_revenue = VALUES(total_revenue),
    avg_order_value = VALUES(avg_order_value);
```

---

## 6. Temporary Table Optimization

### Avoid Disk-Based Temp Tables

```sql
-- Check temp table creation
SHOW STATUS LIKE 'Created_tmp%';
-- Created_tmp_disk_tables should be low

-- Increase limits
SET GLOBAL tmp_table_size = 256 * 1024 * 1024;      -- 256MB
SET GLOBAL max_heap_table_size = 256 * 1024 * 1024; -- 256MB

-- Conditions that force disk-based temp tables:
-- 1. TEXT/BLOB columns in result
-- 2. Result exceeds tmp_table_size
-- 3. UNION (sometimes)
```

### Optimize GROUP BY

```sql
-- Bad: May create temp table
SELECT status, COUNT(*) FROM orders GROUP BY status ORDER BY COUNT(*) DESC;

-- Check EXPLAIN for "Using temporary"

-- Solutions:
-- 1. Add index for GROUP BY column
CREATE INDEX idx_status ON orders (status);

-- 2. Use ORDER BY NULL to skip sorting
SELECT status, COUNT(*) as cnt FROM orders GROUP BY status ORDER BY NULL;
-- Then sort in application if needed
```

---

## 7. Query Cache Strategies

### Application-Level Caching

```sql
-- Instead of relying on query cache (removed in MySQL 8.0)
-- Cache in application layer

-- Identify cacheable queries:
-- 1. Read frequently, written rarely
-- 2. Expensive to compute
-- 3. Stable results

-- Example: Cache product counts per category
-- Redis/Memcached key: product_count:category_123
-- TTL: 5 minutes
-- Invalidate on product insert/update/delete
```

### Materialized Views (Manual)

```sql
-- Create summary table
CREATE TABLE category_product_stats (
    category_id INT PRIMARY KEY,
    product_count INT,
    avg_price DECIMAL(10,2),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Refresh periodically
REPLACE INTO category_product_stats
SELECT
    category_id,
    COUNT(*) as product_count,
    AVG(price) as avg_price,
    NOW()
FROM products
GROUP BY category_id;

-- Query the summary
SELECT * FROM category_product_stats WHERE category_id = 123;
```

---

## 8. Schema Optimization

### Normalize for Writes, Denormalize for Reads

```sql
-- Normalized (good for writes)
SELECT o.id, o.total, u.name, u.email
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending';

-- Denormalized (good for reads)
-- Store user_name, user_email directly in orders
SELECT id, total, user_name, user_email
FROM orders
WHERE status = 'pending';
-- No join needed!

-- Maintain via triggers or application
```

### Proper Data Types

```sql
-- Use smallest data type that fits
-- Bad:
CREATE TABLE items (
    quantity BIGINT,      -- Overkill for item quantity
    price VARCHAR(20),    -- Wrong type for numbers
    status VARCHAR(100)   -- Wasteful for few values
);

-- Good:
CREATE TABLE items (
    quantity SMALLINT UNSIGNED,  -- 0-65535
    price DECIMAL(10,2),         -- Proper numeric
    status ENUM('active', 'inactive', 'deleted')  -- Fixed values
);
```

### Vertical Partitioning

```sql
-- Split wide tables
-- Before:
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    bio TEXT,           -- Rarely accessed
    avatar MEDIUMBLOB,  -- Large, rarely accessed
    settings JSON       -- Rarely accessed
);

-- After:
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    bio TEXT,
    avatar MEDIUMBLOB,
    settings JSON,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Queries on users are faster when not needing profile data
```

---

## 9. Optimizer Hints

### Index Hints

```sql
-- Force index
SELECT * FROM orders FORCE INDEX (idx_status) WHERE status = 'pending';

-- Ignore index
SELECT * FROM orders IGNORE INDEX (idx_created) WHERE status = 'pending';

-- Use index (optimizer can still reject)
SELECT * FROM orders USE INDEX (idx_status) WHERE status = 'pending';
```

### MySQL 8.0 Optimizer Hints

```sql
-- Join strategy
SELECT /*+ BNL(orders) */ * FROM users JOIN orders ON users.id = orders.user_id;
SELECT /*+ HASH_JOIN(orders) */ * FROM users JOIN orders ON users.id = orders.user_id;

-- Disable specific optimizations
SELECT /*+ NO_RANGE_OPTIMIZATION(orders created_at) */ * FROM orders WHERE created_at > '2024-01-01';

-- Set session variables for query
SELECT /*+ SET_VAR(sort_buffer_size = 16M) */ * FROM orders ORDER BY total;

-- Resource group
SELECT /*+ RESOURCE_GROUP(batch_processing) */ * FROM large_table;
```

---

## 10. Optimization Checklist

### Query Analysis Steps

```markdown
1. [ ] Get baseline execution time
2. [ ] Run EXPLAIN / EXPLAIN ANALYZE
3. [ ] Check access type (avoid ALL)
4. [ ] Verify index usage
5. [ ] Check rows examined vs rows returned
6. [ ] Look for filesort/temporary
7. [ ] Consider query rewriting options
8. [ ] Test with proposed indexes
9. [ ] Measure improvement
10. [ ] Document changes
```

### Common Quick Wins

```sql
-- 1. Add missing indexes for WHERE/JOIN columns
-- 2. Convert OR to UNION when beneficial
-- 3. Replace SELECT * with specific columns
-- 4. Add covering indexes for frequent queries
-- 5. Rewrite correlated subqueries as joins
-- 6. Use keyset pagination instead of OFFSET
-- 7. Pre-compute expensive aggregations
```

---

## Summary

| Technique | Use When | Impact |
|-----------|----------|--------|
| Add index | WHERE/JOIN/ORDER BY without index | High |
| Covering index | Frequent queries with few columns | High |
| Query rewrite | Suboptimal query structure | Medium-High |
| Denormalization | Read-heavy, complex joins | Medium |
| Pagination fix | Large OFFSETs | High |
| Pre-aggregation | Expensive repeated aggregations | High |

### Key Principles

1. **Measure first** - Know your baseline
2. **Use EXPLAIN** - Understand the execution plan
3. **Start simple** - Index before rewriting
4. **Test changes** - Verify improvements
5. **Monitor continuously** - Catch regressions early

---

## Further Reading

- "High Performance MySQL" - Query optimization chapters
- MySQL Query Optimization documentation
- Percona Blog - Performance optimization
