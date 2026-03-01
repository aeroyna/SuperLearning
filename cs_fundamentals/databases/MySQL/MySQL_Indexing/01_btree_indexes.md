# B-Tree Indexes in MySQL

## Learning Objectives
- Understand B-Tree index structure and operations
- Learn how InnoDB implements B-Trees
- Master composite index design
- Understand index selectivity and cardinality

---

## 1. B-Tree Structure

### What is a B-Tree?

A B-Tree (Balanced Tree) is a self-balancing tree data structure that maintains sorted data and allows searches, sequential access, insertions, and deletions in O(log n) time.

```
                    B-Tree Structure
                    ================

Level 0 (Root):     ┌───────────────────────────┐
                    │     [50 | 100 | 150]      │
                    └─────────┬────┬────┬───────┘
                              │    │    │
            ┌─────────────────┘    │    └─────────────────┐
            ▼                      ▼                      ▼
Level 1:  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
          │[10|25|40]   │  │[60|75|90]   │  │[120|130|140]│
          └──┬──┬──┬────┘  └──┬──┬──┬────┘  └──┬──┬──┬────┘
             │  │  │          │  │  │          │  │  │
            ...             ...             ...

Level 2 (Leaves):
    ┌───────┐ ┌───────┐ ┌───────┐     ┌───────┐ ┌───────┐
    │1,5,8  │ │11,20  │ │26,30  │ ... │125,127│ │145,148│
    │→Row   │ │→Row   │ │→Row   │     │→Row   │ │→Row   │
    └───────┘ └───────┘ └───────┘     └───────┘ └───────┘
        ↔         ↔         ↔             ↔         ↔
    (Leaf pages are linked for range scans)
```

### B-Tree Properties

- **Balanced**: All leaf nodes at same depth
- **Sorted**: Keys ordered within nodes
- **Fan-out**: Each node holds multiple keys (hundreds in practice)
- **Linked leaves**: Leaf pages linked for range scans

---

## 2. InnoDB B-Tree Implementation

### B+Tree Variant

InnoDB uses B+Tree where:
- All data is in leaf nodes
- Internal nodes contain only keys and pointers
- Leaf nodes are doubly linked

```
┌──────────────────────────────────────────────────────────────────┐
│                     InnoDB B+Tree                                 │
│                                                                   │
│  Internal Nodes:                                                  │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │  [Key1 | Ptr] [Key2 | Ptr] [Key3 | Ptr] ... [KeyN | Ptr]│     │
│  │  (No data, only navigation)                              │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
│  Leaf Nodes (Pages):                                              │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ ← Prev │ [Key|Row] [Key|Row] [Key|Row] ... │ Next →     │     │
│  │        │ (Data stored here)                 │            │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
│  Page Size: 16KB (default)                                        │
│  Typical Fan-out: 200-300 keys per internal node                  │
│  Height for 1M rows: ~3 levels                                    │
└──────────────────────────────────────────────────────────────────┘
```

### Tree Height Calculation

```
Page size: 16KB
Key + pointer: ~15 bytes
Keys per node: ~1000

Height 1: 1,000 rows
Height 2: 1,000 × 1,000 = 1M rows
Height 3: 1,000 × 1,000 × 1,000 = 1B rows

Most tables have B-Tree height of 2-4!
Each level = 1 disk I/O (if not cached)
```

---

## 3. Clustered vs Secondary Indexes

### Clustered Index (Primary Key)

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,  -- Clustered index
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Table data is physically organized by id
-- Looking up by id is fast (direct access)
```

```
Clustered Index Structure:
┌────────────────────────────────────────────────────┐
│  Leaf pages contain COMPLETE rows                   │
│                                                     │
│  Page 1:                                            │
│  ┌──────────────────────────────────────────────┐  │
│  │ id=1 │ name='Alice' │ email='alice@test.com' │  │
│  │ id=2 │ name='Bob'   │ email='bob@test.com'   │  │
│  │ id=3 │ name='Carol' │ email='carol@test.com' │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  Page 2:                                            │
│  ┌──────────────────────────────────────────────┐  │
│  │ id=4 │ name='Dave'  │ email='dave@test.com'  │  │
│  │ id=5 │ name='Eve'   │ email='eve@test.com'   │  │
│  └──────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────┘
```

### Secondary Index

```sql
CREATE INDEX idx_email ON users (email);
```

```
Secondary Index Structure:
┌────────────────────────────────────────────────────┐
│  Leaf pages contain: indexed column + primary key   │
│                                                     │
│  Page 1:                                            │
│  ┌─────────────────────────────────────────┐       │
│  │ email='alice@test.com' │ id=1           │       │
│  │ email='bob@test.com'   │ id=2           │       │
│  │ email='carol@test.com' │ id=3           │       │
│  └─────────────────────────────────────────┘       │
│                                                     │
│  Lookup by email:                                   │
│  1. Find email in secondary index → get id          │
│  2. Look up id in clustered index → get full row   │
│     (This is called a "bookmark lookup")            │
└────────────────────────────────────────────────────┘
```

---

## 4. Composite (Multi-Column) Indexes

### Index Column Order Matters

```sql
CREATE INDEX idx_country_city ON locations (country, city);

-- Index is sorted by country FIRST, then city within each country
```

```
Composite Index Structure:
┌────────────────────────────────────────────────────────────┐
│  Sorted by (country, city)                                  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ ('France', 'Lyon')      │ id=5                       │   │
│  │ ('France', 'Paris')     │ id=2                       │   │
│  │ ('Germany', 'Berlin')   │ id=3                       │   │
│  │ ('Germany', 'Munich')   │ id=7                       │   │
│  │ ('USA', 'Boston')       │ id=6                       │   │
│  │ ('USA', 'Chicago')      │ id=4                       │   │
│  │ ('USA', 'New York')     │ id=1                       │   │
│  └─────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### Leftmost Prefix Rule

```sql
INDEX idx_a_b_c (a, b, c)

-- Can use index for:
WHERE a = 1                      -- ✓ Uses index
WHERE a = 1 AND b = 2            -- ✓ Uses index
WHERE a = 1 AND b = 2 AND c = 3  -- ✓ Uses index (full)
WHERE a = 1 AND c = 3            -- ✓ Uses a, skips b, can't use c

-- Cannot use index for:
WHERE b = 2                      -- ✗ Can't skip 'a'
WHERE c = 3                      -- ✗ Can't skip 'a' and 'b'
WHERE b = 2 AND c = 3            -- ✗ Can't skip 'a'
```

### Column Order Strategy

```sql
-- Put most selective column first? Not always!

-- Consider query patterns:
-- Query 1: WHERE status = 'active' AND created_at > '2024-01-01'
-- Query 2: WHERE status = 'active'
-- Query 3: WHERE created_at > '2024-01-01'

-- If Query 2 is most common:
CREATE INDEX idx_status_created ON orders (status, created_at);

-- Equality conditions before range conditions:
-- status = 'active' (equality) before created_at > date (range)
```

---

## 5. Index Operations

### Index Lookup

```sql
-- Point lookup: O(log n)
SELECT * FROM users WHERE id = 100;

EXPLAIN:
┌────────────────────────────────────────────────────────────────┐
│ type: const                                                     │
│ key: PRIMARY                                                    │
│ rows: 1                                                         │
│ Extra: NULL                                                     │
└────────────────────────────────────────────────────────────────┘
```

### Range Scan

```sql
-- Range scan: O(log n + k) where k = matching rows
SELECT * FROM orders WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';

EXPLAIN:
┌────────────────────────────────────────────────────────────────┐
│ type: range                                                     │
│ key: idx_created_at                                             │
│ rows: 15000                                                     │
│ Extra: Using index condition                                    │
└────────────────────────────────────────────────────────────────┘
```

### Index Scan vs Table Scan

```sql
-- Full index scan (reading entire index)
SELECT * FROM users ORDER BY email;

EXPLAIN:
┌────────────────────────────────────────────────────────────────┐
│ type: index                                                     │
│ key: idx_email                                                  │
│ rows: 100000                                                    │
│ Extra: NULL                                                     │
└────────────────────────────────────────────────────────────────┘

-- Full table scan (reading entire table)
SELECT * FROM users;

EXPLAIN:
┌────────────────────────────────────────────────────────────────┐
│ type: ALL                                                       │
│ key: NULL                                                       │
│ rows: 100000                                                    │
│ Extra: NULL                                                     │
└────────────────────────────────────────────────────────────────┘
```

---

## 6. Index Selectivity and Cardinality

### Cardinality

Number of unique values in an index.

```sql
-- Check cardinality
SHOW INDEX FROM users;

+-------+------------+----------+--------------+-------------+-----------+-------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality |
+-------+------------+----------+--------------+-------------+-----------+-------------+
| users |          0 | PRIMARY  |            1 | id          | A         |      100000 |
| users |          1 | idx_email|            1 | email       | A         |      100000 |
| users |          1 | idx_status|           1 | status      | A         |           3 |
+-------+------------+----------+--------------+-------------+-----------+-------------+

-- email: High cardinality (100K unique) - Good for index
-- status: Low cardinality (3 values) - Poor for standalone index
```

### Selectivity

```sql
-- Selectivity = Cardinality / Total Rows

-- email selectivity: 100000 / 100000 = 1.0 (excellent)
-- status selectivity: 3 / 100000 = 0.00003 (poor)

-- Good selectivity: > 0.1 (10%)
-- Excellent selectivity: > 0.9 (90%)
```

### Low Cardinality Index Use Cases

```sql
-- Low cardinality alone is usually bad:
CREATE INDEX idx_status ON users (status);
SELECT * FROM users WHERE status = 'active';  -- 80% of rows, full scan faster

-- But combined with other columns can be useful:
CREATE INDEX idx_status_created ON users (status, created_at);
SELECT * FROM users WHERE status = 'active' AND created_at > '2024-01-01';
```

---

## 7. Index Maintenance

### Update Statistics

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
```

### Index Fragmentation

```sql
-- Check fragmentation
SELECT
    TABLE_NAME,
    INDEX_NAME,
    STAT_VALUE AS pages,
    (SELECT STAT_VALUE FROM mysql.innodb_index_stats s2
     WHERE s2.database_name = s1.database_name
       AND s2.table_name = s1.table_name
       AND s2.index_name = s1.index_name
       AND s2.stat_name = 'n_leaf_pages') AS leaf_pages
FROM mysql.innodb_index_stats s1
WHERE database_name = 'mydb'
  AND stat_name = 'size';

-- Rebuild fragmented index
ALTER TABLE users ENGINE = InnoDB;

-- Or optimize specific table
OPTIMIZE TABLE users;
```

---

## 8. B-Tree Index Limitations

### What B-Trees Cannot Do Efficiently

```sql
-- 1. Leading wildcard searches
SELECT * FROM users WHERE email LIKE '%@gmail.com';  -- Full scan
SELECT * FROM users WHERE email LIKE 'john%';        -- Can use index

-- 2. Functions on indexed columns
SELECT * FROM users WHERE YEAR(created_at) = 2024;   -- Full scan
SELECT * FROM users WHERE created_at >= '2024-01-01'
                      AND created_at < '2025-01-01';  -- Uses index

-- 3. OR conditions on different columns
SELECT * FROM users WHERE email = 'a@b.com' OR name = 'John';
-- Need separate indexes and index merge, or UNION

-- 4. Not-equal conditions
SELECT * FROM orders WHERE status != 'cancelled';  -- Usually full scan
```

### Solutions

```sql
-- 1. Full-text index for substring search
CREATE FULLTEXT INDEX ft_email ON users (email);
SELECT * FROM users WHERE MATCH(email) AGAINST('gmail.com');

-- 2. Generated columns for function results
ALTER TABLE users ADD COLUMN created_year INT
    GENERATED ALWAYS AS (YEAR(created_at)) STORED;
CREATE INDEX idx_year ON users (created_year);

-- 3. Multiple indexes with UNION
SELECT * FROM users WHERE email = 'a@b.com'
UNION
SELECT * FROM users WHERE name = 'John';
```

---

## 9. Practical Examples

### E-commerce Product Search

```sql
-- Products table
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    category_id INT,
    price DECIMAL(10,2),
    stock INT,
    created_at DATETIME,
    is_active BOOLEAN
);

-- Common queries and indexes:

-- 1. Browse by category with price filter
CREATE INDEX idx_category_price ON products (category_id, price);
SELECT * FROM products WHERE category_id = 5 AND price < 100;

-- 2. Search active products by category
CREATE INDEX idx_category_active ON products (category_id, is_active);
SELECT * FROM products WHERE category_id = 5 AND is_active = TRUE;

-- 3. Admin: recent products
CREATE INDEX idx_created ON products (created_at);
SELECT * FROM products ORDER BY created_at DESC LIMIT 20;

-- 4. Inventory check
CREATE INDEX idx_stock ON products (stock);
SELECT * FROM products WHERE stock < 10;
```

### User Session Management

```sql
-- Sessions table
CREATE TABLE sessions (
    id CHAR(36) PRIMARY KEY,  -- UUID
    user_id INT,
    created_at DATETIME,
    expires_at DATETIME,
    ip_address VARCHAR(45),
    INDEX idx_user (user_id),
    INDEX idx_expires (expires_at)
);

-- Lookup user sessions
SELECT * FROM sessions WHERE user_id = 123;

-- Cleanup expired sessions
DELETE FROM sessions WHERE expires_at < NOW();
```

---

## Summary

| Concept | Description |
|---------|-------------|
| B-Tree | Self-balancing tree, O(log n) operations |
| Clustered Index | Primary key, contains full row data |
| Secondary Index | Contains indexed columns + primary key |
| Leftmost Prefix | Must use leftmost columns of composite index |
| Cardinality | Number of unique values |
| Selectivity | Ratio of unique values to total rows |

---

## Further Reading

- "Database Internals" by Alex Petrov - B-Tree chapter
- MySQL Index documentation
- "Use The Index, Luke" - indexing tutorial
