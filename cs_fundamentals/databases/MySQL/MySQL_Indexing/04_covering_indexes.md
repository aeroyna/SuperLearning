# Covering Indexes in MySQL

## Learning Objectives
- Understand what covering indexes are
- Learn how to identify opportunities for covering indexes
- Master index-only query optimization
- Design efficient composite indexes

---

## 1. What is a Covering Index?

A covering index contains all columns needed to satisfy a query, eliminating the need to access the actual table data.

```
┌─────────────────────────────────────────────────────────────────────┐
│            Regular Index Lookup vs Covering Index                    │
│                                                                      │
│  Query: SELECT email, name FROM users WHERE email = 'a@test.com'     │
│                                                                      │
│  Regular Index (idx_email):                                          │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Step 1: Search secondary index                                   ││
│  │ ┌─────────────────┐                                              ││
│  │ │ email index     │ → email='a@test.com' → PK=5                  ││
│  │ └─────────────────┘                                              ││
│  │                                                                   ││
│  │ Step 2: Lookup in clustered index (bookmark lookup)              ││
│  │ ┌─────────────────┐                                              ││
│  │ │ Clustered Index │ → PK=5 → [id=5, email='a@test.com',         ││
│  │ │                 │           name='Alice', other columns...]    ││
│  │ └─────────────────┘                                              ││
│  │                                                                   ││
│  │ Total: 2 index lookups (secondary + clustered)                   ││
│  └─────────────────────────────────────────────────────────────────┘│
│                                                                      │
│  Covering Index (idx_email_name):                                    │
│  ┌─────────────────────────────────────────────────────────────────┐│
│  │ Step 1: Search composite index - DONE!                           ││
│  │ ┌─────────────────────────────────────────────┐                  ││
│  │ │ (email, name) index                         │                  ││
│  │ │ → email='a@test.com' → name='Alice'        │                  ││
│  │ │ (All needed data is in the index!)          │                  ││
│  │ └─────────────────────────────────────────────┘                  ││
│  │                                                                   ││
│  │ Total: 1 index lookup only (no table access)                     ││
│  └─────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Identifying Covering Index Usage

### EXPLAIN Output

```sql
-- Check for "Using index" in Extra column
EXPLAIN SELECT email, status FROM users WHERE email = 'test@example.com';

+----+-------------+-------+------+---------------+----------+---------+-------+------+-------------+
| id | select_type | table | type | possible_keys | key      | key_len | ref   | rows | Extra       |
+----+-------------+-------+------+---------------+----------+---------+-------+------+-------------+
|  1 | SIMPLE      | users | ref  | idx_email_status | idx_email_status | 303 | const |    1 | Using index |
+----+-------------+-------+------+---------------+----------+---------+-------+------+-------------+

-- "Using index" = Covering index is being used
-- No table data access needed!
```

### What the EXPLAIN Output Means

```sql
-- "Using index" → Covering index (index-only scan)
-- "Using where" → Filter applied after reading
-- "Using index condition" → Index Condition Pushdown (ICP)
-- NULL in Extra → Rows fetched from table

-- Best case: "Using index" only
-- Good case: "Using index; Using where"
-- Needs optimization: Neither "Using index"
```

---

## 3. Creating Covering Indexes

### Basic Covering Index

```sql
-- Query to optimize:
SELECT email, name, created_at
FROM users
WHERE email = 'test@example.com';

-- Create covering index with all needed columns
CREATE INDEX idx_covering ON users (email, name, created_at);

-- Now the query uses index-only scan
EXPLAIN SELECT email, name, created_at
FROM users WHERE email = 'test@example.com';
-- Extra: Using index
```

### Column Order Strategy

```sql
-- Rule: Put WHERE columns first, then SELECT columns

-- Query:
SELECT status, created_at
FROM orders
WHERE user_id = 123
ORDER BY created_at DESC;

-- Optimal covering index:
CREATE INDEX idx_user_created_status ON orders (user_id, created_at DESC, status);

-- This index:
-- 1. Filters by user_id (equality)
-- 2. Already sorted by created_at (no filesort)
-- 3. Includes status (covering)
```

### Including All Query Columns

```sql
-- Original query and index:
SELECT product_id, quantity, price
FROM order_items
WHERE order_id = 1001;

-- Non-covering index:
CREATE INDEX idx_order ON order_items (order_id);
-- Needs to lookup clustered index for product_id, quantity, price

-- Covering index:
CREATE INDEX idx_order_covering ON order_items (order_id, product_id, quantity, price);
-- All data in index!

-- Primary key is always included in secondary indexes
-- So if 'id' is PK, no need to add it explicitly
```

---

## 4. InnoDB Covering Index Behavior

### Secondary Indexes Include Primary Key

```sql
-- In InnoDB, secondary index automatically includes PK columns
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100),
    name VARCHAR(100),
    INDEX idx_email (email)
);

-- idx_email actually stores: (email, id)
-- So this query is covered:
SELECT id FROM users WHERE email = 'test@example.com';
-- Extra: Using index
```

### Composite Primary Key Considerations

```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)
);

CREATE INDEX idx_product ON order_items (product_id);

-- idx_product actually stores: (product_id, order_id)
-- Both PK columns are included!

-- Covered query:
SELECT order_id, product_id FROM order_items WHERE product_id = 100;
-- Extra: Using index
```

---

## 5. Advanced Covering Index Patterns

### Covering Index for Aggregations

```sql
-- Query: Get order count per user
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id;

-- Covering index:
CREATE INDEX idx_user_covering ON orders (user_id);

-- The index has all data needed for GROUP BY and COUNT
-- Extra: Using index
```

### Covering Index for JOINs

```sql
-- Query joining orders and users
SELECT o.id, o.total, u.email
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending';

-- For orders table:
CREATE INDEX idx_status_user_id_total ON orders (status, user_id, id, total);

-- For users table (lookup by id - clustered index handles this)
-- But if we select more columns from users:
CREATE INDEX idx_user_email ON users (id, email);

-- Both indexes can be covering for their respective tables
```

### Covering Index for Subqueries

```sql
-- Query with subquery
SELECT * FROM products
WHERE category_id IN (
    SELECT id FROM categories WHERE active = TRUE
);

-- Covering index for subquery:
CREATE INDEX idx_active_id ON categories (active, id);

-- Subquery uses index-only scan
```

---

## 6. Trade-offs and Considerations

### Storage Overhead

```sql
-- More columns = larger index = more storage

-- Check index sizes
SELECT
    TABLE_NAME,
    INDEX_NAME,
    ROUND(STAT_VALUE * @@innodb_page_size / 1024 / 1024, 2) AS size_mb
FROM mysql.innodb_index_stats
WHERE stat_name = 'size'
AND database_name = 'mydb'
ORDER BY STAT_VALUE DESC;

-- Balance: Don't include rarely-used columns
-- Wide covering indexes waste space if not frequently used
```

### Write Performance Impact

```sql
-- Every INSERT/UPDATE must update all indexes
-- More columns in index = more data to write

-- Consider write frequency:
-- - Read-heavy table: Covering indexes beneficial
-- - Write-heavy table: Minimize index columns

-- Test with your workload!
```

### When NOT to Use Covering Indexes

```sql
-- 1. SELECT * queries (too many columns)
SELECT * FROM users WHERE email = 'test@example.com';
-- Better: Select only needed columns, create covering index

-- 2. Very wide columns
-- Avoid including TEXT, BLOB, or large VARCHAR in covering index
-- Index size limit: ~3072 bytes for InnoDB

-- 3. Rarely executed queries
-- Not worth the storage and write overhead

-- 4. Tables with many write operations
-- Each additional index column slows inserts/updates
```

---

## 7. Optimization Examples

### E-Commerce Product Listing

```sql
-- Common query: List products in category
SELECT id, name, price, image_url
FROM products
WHERE category_id = 5
  AND in_stock = TRUE
ORDER BY price ASC
LIMIT 20;

-- Covering index:
CREATE INDEX idx_category_stock_price_covering
ON products (category_id, in_stock, price, id, name, image_url);

-- Analysis:
-- category_id: Equality filter (first)
-- in_stock: Equality filter (second)
-- price: ORDER BY (third, can use index for sorting)
-- id, name, image_url: SELECT columns (covering)
```

### User Activity Dashboard

```sql
-- Query: Recent activity summary
SELECT user_id, action, created_at
FROM activity_log
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 24 HOUR)
ORDER BY created_at DESC
LIMIT 100;

-- Covering index:
CREATE INDEX idx_time_covering ON activity_log (created_at DESC, user_id, action);

-- Uses index for:
-- - Range condition (created_at)
-- - ORDER BY (already sorted)
-- - SELECT columns (covering)
```

### Reporting Query

```sql
-- Query: Daily sales summary
SELECT
    DATE(order_date) as day,
    COUNT(*) as order_count,
    SUM(total) as revenue
FROM orders
WHERE order_date >= '2024-01-01'
  AND status = 'completed'
GROUP BY DATE(order_date);

-- Covering index:
CREATE INDEX idx_report ON orders (status, order_date, total);

-- This covers:
-- - status filter
-- - order_date range and grouping
-- - total for SUM aggregation
```

---

## 8. Monitoring Covering Index Effectiveness

### Query Performance

```sql
-- Compare execution times
SET profiling = 1;

SELECT email, name FROM users WHERE email = 'test@example.com';

SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;

-- Look for reduction in:
-- - Sending data time
-- - Handler_read_rnd_next (random reads)
```

### Handler Statistics

```sql
-- Check before and after adding covering index
FLUSH STATUS;

SELECT email, name FROM users WHERE email = 'test@example.com';

SHOW STATUS LIKE 'Handler%';

-- Key metrics:
-- Handler_read_key: Index lookups (should be low)
-- Handler_read_next: Sequential index reads
-- Handler_read_rnd_next: Table scans (should be 0 for covering index)
```

### Index Usage Statistics

```sql
-- Check which indexes are being used (MySQL 8.0)
SELECT
    OBJECT_SCHEMA,
    OBJECT_NAME,
    INDEX_NAME,
    COUNT_READ,
    COUNT_FETCH
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE OBJECT_SCHEMA = 'mydb'
ORDER BY COUNT_READ DESC;
```

---

## 9. Practical Checklist

### Creating Effective Covering Indexes

```sql
-- Step 1: Identify the query to optimize
SELECT email, status, created_at
FROM users
WHERE email = 'test@example.com';

-- Step 2: List all columns used
-- WHERE: email
-- SELECT: email, status, created_at
-- ORDER BY: (none)
-- GROUP BY: (none)

-- Step 3: Order columns
-- 1. Equality conditions (WHERE =)
-- 2. Range conditions (WHERE <, >, BETWEEN)
-- 3. ORDER BY columns
-- 4. GROUP BY columns
-- 5. SELECT columns (for covering)

-- Step 4: Create index
CREATE INDEX idx_covering ON users (email, status, created_at);

-- Step 5: Verify with EXPLAIN
EXPLAIN SELECT email, status, created_at
FROM users WHERE email = 'test@example.com';
-- Should show: Using index
```

---

## Summary

| Aspect | Covering Index | Regular Index |
|--------|----------------|---------------|
| Data Access | Index only | Index + Table |
| I/O Operations | Minimal | More (bookmark lookup) |
| Storage | Larger | Smaller |
| Write Speed | Slower | Faster |
| EXPLAIN Extra | "Using index" | NULL or "Using where" |

### Key Takeaways

1. **Covering indexes** eliminate table lookups
2. **Column order matters**: filters → sort → select
3. **InnoDB includes PK** in all secondary indexes
4. **Trade-off**: Storage and write speed vs read speed
5. **Monitor with EXPLAIN**: Look for "Using index"

---

## Further Reading

- "High Performance MySQL" - Indexing Strategies
- MySQL Index Optimization documentation
- "Use The Index, Luke" - Covering Indexes chapter
