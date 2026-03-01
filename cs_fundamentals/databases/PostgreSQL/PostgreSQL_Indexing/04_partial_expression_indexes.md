# Partial and Expression Indexes

## Learning Objectives
- Create partial indexes for selective queries
- Use expression indexes for computed values
- Optimize specific query patterns
- Reduce index size and maintenance cost

---

## 1. Partial Indexes

### What is a Partial Index?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Partial Index Concept                             │
│                                                                      │
│  Full Index (indexes ALL rows):                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Table: 10 million rows                                       │    │
│  │ Index: 10 million entries                                    │    │
│  │ Queries: SELECT * WHERE status = 'pending'                   │    │
│  │          (but 'pending' is only 1% of data)                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Partial Index (indexes SUBSET of rows):                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Table: 10 million rows                                       │    │
│  │ Index: 100,000 entries (only 'pending' rows)                 │    │
│  │ Smaller, faster, less maintenance                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  CREATE INDEX idx_pending ON orders (id) WHERE status = 'pending';  │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating Partial Indexes

```sql
-- Basic partial index
CREATE INDEX idx_orders_pending ON orders (created_at)
WHERE status = 'pending';

-- Only indexes rows where status = 'pending'
-- Much smaller than full index

-- Query must match the WHERE clause to use index:
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2024-01-01';
-- ✓ Uses idx_orders_pending

SELECT * FROM orders WHERE status = 'shipped' AND created_at > '2024-01-01';
-- ✗ Cannot use idx_orders_pending
```

### Common Partial Index Patterns

```sql
-- Active records only
CREATE INDEX idx_users_active_email ON users (email)
WHERE active = true;

-- Non-null values
CREATE INDEX idx_users_phone ON users (phone)
WHERE phone IS NOT NULL;

-- Specific categories
CREATE INDEX idx_products_electronics ON products (name)
WHERE category = 'electronics';

-- Recent data
CREATE INDEX idx_logs_recent ON logs (created_at)
WHERE created_at > '2024-01-01';
-- Note: Static date - may need periodic recreation

-- Multiple conditions
CREATE INDEX idx_orders_open ON orders (customer_id, created_at)
WHERE status IN ('pending', 'processing') AND deleted_at IS NULL;
```

### Unique Partial Indexes

```sql
-- Unique constraint on subset
CREATE UNIQUE INDEX idx_users_email_active ON users (email)
WHERE active = true;

-- Allows multiple inactive users with same email
-- Only active users must have unique email

-- Soft delete pattern
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    slug VARCHAR(100),
    title VARCHAR(200),
    deleted_at TIMESTAMP
);

CREATE UNIQUE INDEX idx_articles_slug ON articles (slug)
WHERE deleted_at IS NULL;

-- Active articles have unique slugs
-- Deleted articles can have duplicate slugs
```

### Partial Index Size Comparison

```sql
-- Compare sizes
-- Full index
CREATE INDEX idx_orders_full ON orders (customer_id);

-- Partial index (10% of data)
CREATE INDEX idx_orders_partial ON orders (customer_id)
WHERE status = 'pending';

-- Check sizes
SELECT
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS size
FROM pg_indexes
WHERE tablename = 'orders';

-- Partial index can be 10x smaller!
```

---

## 2. Expression Indexes

### What is an Expression Index?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Expression Index Concept                          │
│                                                                      │
│  Problem: Query with function on column                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ SELECT * FROM users WHERE LOWER(email) = 'john@example.com'  │    │
│  │                                                               │    │
│  │ Regular index on email cannot be used!                        │    │
│  │ Must evaluate LOWER(email) for every row → full table scan   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Solution: Index the expression                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ CREATE INDEX idx_users_email_lower ON users (LOWER(email));  │    │
│  │                                                               │    │
│  │ Index stores: lower(email) → row                              │    │
│  │ Query now uses index for LOWER(email) lookups                │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating Expression Indexes

```sql
-- Case-insensitive search
CREATE INDEX idx_users_email_lower ON users (LOWER(email));

-- Query must use same expression:
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';  -- ✓ Uses index
SELECT * FROM users WHERE email = 'john@example.com';         -- ✗ Different

-- Date extraction
CREATE INDEX idx_orders_year ON orders (EXTRACT(YEAR FROM created_at));

SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;

-- JSON field extraction
CREATE INDEX idx_events_user ON events ((data->>'user_id'));

SELECT * FROM events WHERE data->>'user_id' = '12345';
```

### Common Expression Index Patterns

```sql
-- Text normalization
CREATE INDEX idx_products_name_normalized ON products (
    LOWER(TRIM(name))
);

-- Computed values
CREATE INDEX idx_orders_total ON orders (
    (quantity * unit_price)
);

SELECT * FROM orders WHERE quantity * unit_price > 1000;

-- Substring/pattern
CREATE INDEX idx_phones_area ON contacts (
    SUBSTRING(phone FROM 1 FOR 3)
);

SELECT * FROM contacts WHERE SUBSTRING(phone FROM 1 FOR 3) = '212';

-- Date truncation
CREATE INDEX idx_logs_day ON logs (
    DATE_TRUNC('day', created_at)
);

SELECT COUNT(*) FROM logs
WHERE DATE_TRUNC('day', created_at) = '2024-01-15';

-- JSONB nested field
CREATE INDEX idx_users_country ON users (
    (profile->'address'->>'country')
);
```

### Expression Index Requirements

```sql
-- Expression must be IMMUTABLE
-- This works:
CREATE INDEX idx_lower ON users (LOWER(email));
-- LOWER is immutable (same input always gives same output)

-- This fails:
CREATE INDEX idx_age ON users (AGE(birth_date));
-- AGE depends on current date (not immutable)

-- Workaround for non-immutable:
CREATE INDEX idx_birth_year ON users (EXTRACT(YEAR FROM birth_date));
-- EXTRACT(YEAR ...) is immutable

-- Check function volatility:
SELECT proname, provolatile FROM pg_proc WHERE proname = 'lower';
-- 'i' = immutable, 's' = stable, 'v' = volatile
```

---

## 3. Combining Partial and Expression

### Powerful Combinations

```sql
-- Case-insensitive search on active users only
CREATE INDEX idx_users_email_ci_active ON users (LOWER(email))
WHERE active = true;

-- JSONB field on recent orders
CREATE INDEX idx_orders_metadata ON orders ((data->>'priority'))
WHERE created_at > '2024-01-01';

-- Computed value on non-deleted records
CREATE INDEX idx_products_margin ON products ((price - cost))
WHERE deleted_at IS NULL AND price > cost;
```

### Multi-Column Expression Index

```sql
-- Full name search
CREATE INDEX idx_users_fullname ON users (
    LOWER(first_name || ' ' || last_name)
);

SELECT * FROM users
WHERE LOWER(first_name || ' ' || last_name) LIKE 'john doe%';

-- Combined with partial
CREATE INDEX idx_users_fullname_active ON users (
    LOWER(first_name || ' ' || last_name)
)
WHERE active = true;
```

---

## 4. Query Matching

### Understanding Index Selection

```sql
-- Index definition
CREATE INDEX idx_orders_pending ON orders (customer_id)
WHERE status = 'pending';

-- These queries USE the partial index:
SELECT * FROM orders WHERE customer_id = 100 AND status = 'pending';
SELECT * FROM orders WHERE status = 'pending' AND customer_id IN (1, 2, 3);

-- These queries DO NOT USE the partial index:
SELECT * FROM orders WHERE customer_id = 100;
-- Missing status = 'pending'

SELECT * FROM orders WHERE customer_id = 100 AND status = 'shipped';
-- Different status value

SELECT * FROM orders WHERE customer_id = 100 AND status IN ('pending', 'shipped');
-- Condition doesn't imply WHERE clause
```

### Expression Matching

```sql
-- Index definition
CREATE INDEX idx_users_email_lower ON users (LOWER(email));

-- MUST use exact same expression:
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';  -- ✓
SELECT * FROM users WHERE email = 'test@example.com';          -- ✗
SELECT * FROM users WHERE UPPER(email) = 'TEST@EXAMPLE.COM';   -- ✗
SELECT * FROM users WHERE lower(email) = 'test@example.com';   -- ✓ (case-insensitive)
```

### Verifying Index Usage

```sql
-- Always check with EXPLAIN
EXPLAIN SELECT * FROM orders WHERE status = 'pending' AND customer_id = 100;

-- Look for:
-- Index Scan using idx_orders_pending
-- NOT: Seq Scan on orders

-- With ANALYZE for actual execution
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending' AND customer_id = 100;
```

---

## 5. Practical Examples

### Soft Delete Pattern

```sql
-- Table with soft deletes
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    owner_id INTEGER,
    deleted_at TIMESTAMP
);

-- Index only non-deleted documents
CREATE INDEX idx_docs_owner ON documents (owner_id)
WHERE deleted_at IS NULL;

-- Unique title per owner (for non-deleted)
CREATE UNIQUE INDEX idx_docs_owner_title ON documents (owner_id, title)
WHERE deleted_at IS NULL;

-- Query pattern
SELECT * FROM documents
WHERE owner_id = 100
  AND deleted_at IS NULL
ORDER BY title;
```

### Multi-Tenant Application

```sql
-- Tenant-specific indexes
CREATE TABLE tenant_data (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    category VARCHAR(50),
    data JSONB
);

-- High-traffic tenant gets dedicated index
CREATE INDEX idx_tenant_1_category ON tenant_data (category)
WHERE tenant_id = 1;

-- Other tenants share general index
CREATE INDEX idx_tenant_other ON tenant_data (tenant_id, category)
WHERE tenant_id != 1;
```

### Date-Based Hot Data

```sql
-- Recent data accessed frequently
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(50),
    user_id INTEGER,
    created_at TIMESTAMP DEFAULT NOW(),
    processed BOOLEAN DEFAULT FALSE
);

-- Index only unprocessed events
CREATE INDEX idx_events_unprocessed ON events (event_type, user_id)
WHERE processed = FALSE;

-- Index recent events for analytics
CREATE INDEX idx_events_recent ON events (event_type, created_at DESC)
WHERE created_at > NOW() - INTERVAL '30 days';
-- Note: This static expression won't update automatically
-- Consider periodic recreation or use BRIN for historical data

-- Better approach with function:
CREATE OR REPLACE FUNCTION is_recent(ts TIMESTAMP) RETURNS BOOLEAN
IMMUTABLE LANGUAGE SQL AS $$
    SELECT ts > '2024-01-01'::timestamp;  -- Update periodically
$$;

CREATE INDEX idx_events_recent ON events (event_type, created_at)
WHERE is_recent(created_at);
```

### Search Optimization

```sql
-- E-commerce product search
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    sku VARCHAR(50),
    category_id INTEGER,
    brand_id INTEGER,
    price NUMERIC(10,2),
    in_stock BOOLEAN DEFAULT TRUE,
    search_vector TSVECTOR
);

-- Case-insensitive SKU lookup (in stock only)
CREATE UNIQUE INDEX idx_products_sku ON products (UPPER(sku))
WHERE in_stock = TRUE;

-- Price range by category (in stock)
CREATE INDEX idx_products_cat_price ON products (category_id, price)
WHERE in_stock = TRUE;

-- Full-text search (in stock)
CREATE INDEX idx_products_search ON products USING GIN (search_vector)
WHERE in_stock = TRUE;
```

---

## 6. Maintenance Considerations

### Index Overhead

```sql
-- Partial indexes reduce maintenance overhead
-- Only affected rows update the index

-- Full index: Every insert/update/delete updates index
-- Partial index: Only matching rows update index

-- Monitor index usage
SELECT
    indexrelname,
    idx_scan AS scans,
    idx_tup_read AS tuples_read,
    idx_tup_fetch AS tuples_fetched,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;
```

### Identifying Candidates

```sql
-- Find columns with skewed distribution
SELECT
    attname,
    null_frac,
    n_distinct,
    most_common_vals,
    most_common_freqs
FROM pg_stats
WHERE tablename = 'orders'
  AND attname = 'status';

-- If one value dominates, consider partial index
-- Example: 95% 'completed', 5% 'pending'
-- Create partial index for 'pending' (the minority frequently queried)
```

### When NOT to Use

```sql
-- Avoid partial indexes when:
-- 1. Query patterns vary widely
-- 2. WHERE clause values change frequently
-- 3. Partial set is large (> 50% of table)
-- 4. Queries need both included and excluded rows

-- Bad: Partial index on large subset
CREATE INDEX idx_orders_not_cancelled ON orders (customer_id)
WHERE status != 'cancelled';  -- If 95% not cancelled, not helpful

-- Good: Partial index on small subset
CREATE INDEX idx_orders_cancelled ON orders (customer_id)
WHERE status = 'cancelled';   -- If 5% cancelled, very helpful
```

---

## Summary

| Feature | Partial Index | Expression Index |
|---------|---------------|------------------|
| Purpose | Index subset of rows | Index computed values |
| Syntax | `WHERE condition` | `(expression)` |
| Size | Smaller than full | Same as full |
| Use case | Skewed data, hot data | Case-insensitive, JSON |
| Maintenance | Only matching rows | All rows |

---

## Further Reading

- PostgreSQL Indexes documentation
- CREATE INDEX documentation
- Query Planning with indexes
