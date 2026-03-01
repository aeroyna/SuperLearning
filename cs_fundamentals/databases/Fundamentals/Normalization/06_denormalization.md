# Denormalization

## 1. Introduction

**Denormalization** is the deliberate introduction of redundancy into a database design to improve read performance. It's the opposite of normalization.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DENORMALIZATION OVERVIEW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Normalization: Optimize for data integrity and minimal redundancy         │
│   Denormalization: Optimize for read performance                            │
│                                                                              │
│   ┌─────────────┐                      ┌─────────────┐                      │
│   │ Normalized  │     TRADE-OFFS       │Denormalized │                      │
│   ├─────────────┤    ◄───────────►     ├─────────────┤                      │
│   │ Less storage│                      │ More storage│                      │
│   │ Slower reads│                      │ Faster reads│                      │
│   │ Fast writes │                      │ Slower writes│                     │
│   │ Consistency │                      │ Risk of     │                      │
│   │ guaranteed  │                      │ inconsistency│                     │
│   └─────────────┘                      └─────────────┘                      │
│                                                                              │
│   Rule: Normalize first, denormalize only when needed                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. When to Denormalize

### 2.1 Valid Reasons

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WHEN TO DENORMALIZE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ✓ READ-HEAVY WORKLOADS                                                    │
│     • Reports and analytics                                                 │
│     • Dashboard queries                                                     │
│     • Read ratio >> Write ratio (e.g., 100:1)                              │
│                                                                              │
│   ✓ QUERY PERFORMANCE IS CRITICAL                                          │
│     • User-facing queries with strict SLAs                                 │
│     • Complex joins causing timeouts                                        │
│     • Measured performance issues (not premature optimization!)            │
│                                                                              │
│   ✓ DATA RARELY CHANGES                                                     │
│     • Historical/archival data                                             │
│     • Reference data that's mostly static                                  │
│                                                                              │
│   ✓ SPECIFIC QUERY PATTERNS                                                 │
│     • Known, predictable access patterns                                   │
│     • Same data often queried together                                     │
│                                                                              │
│   ✗ AVOID IF:                                                               │
│     • Write-heavy workloads                                                │
│     • Data changes frequently                                              │
│     • Consistency is critical                                              │
│     • Just "seems faster" without measurement                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Denormalization Techniques

### 3.1 Storing Derived/Calculated Values

```sql
-- BEFORE: Calculated at query time
SELECT
    o.order_id,
    o.customer_id,
    (SELECT SUM(quantity * unit_price) FROM order_items WHERE order_id = o.order_id) AS total
FROM orders o;

-- AFTER: Pre-calculated and stored
ALTER TABLE orders ADD COLUMN total DECIMAL(12, 2);

-- Update when order items change
UPDATE orders o
SET total = (SELECT SUM(quantity * unit_price) FROM order_items WHERE order_id = o.order_id)
WHERE order_id = :order_id;

-- Fast query
SELECT order_id, customer_id, total FROM orders;
```

### 3.2 Duplicating Columns from Related Tables

```sql
-- BEFORE: Join required
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    c.email
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- AFTER: Customer info duplicated in orders
ALTER TABLE orders ADD COLUMN customer_name VARCHAR(100);
ALTER TABLE orders ADD COLUMN customer_email VARCHAR(255);

-- Fast query (no join)
SELECT order_id, order_date, customer_name, customer_email FROM orders;

-- Trade-off: Must update orders when customer info changes
-- Consider: Is customer_name at ORDER TIME what we want? (historical accuracy)
```

### 3.3 Pre-joined Tables

```sql
-- BEFORE: Multiple joins for reporting
SELECT
    o.order_id,
    o.order_date,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;

-- AFTER: Denormalized reporting table
CREATE TABLE order_details_denorm (
    order_id INT,
    order_date DATE,
    customer_id INT,
    customer_name VARCHAR(100),
    product_id INT,
    product_name VARCHAR(100),
    quantity INT,
    unit_price DECIMAL(10, 2),
    line_total DECIMAL(12, 2)
);

-- Populate via ETL or triggers
-- Query is now a simple SELECT with no joins
```

### 3.4 Summary/Aggregate Tables

```sql
-- BEFORE: Aggregation at query time
SELECT
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY DATE_TRUNC('month', order_date);

-- AFTER: Pre-aggregated summary table
CREATE TABLE monthly_sales_summary (
    year_month DATE PRIMARY KEY,
    order_count INT,
    revenue DECIMAL(15, 2),
    avg_order_value DECIMAL(10, 2),
    updated_at TIMESTAMP
);

-- Update periodically or via triggers
INSERT INTO monthly_sales_summary (year_month, order_count, revenue, avg_order_value, updated_at)
SELECT
    DATE_TRUNC('month', order_date),
    COUNT(*),
    SUM(total),
    AVG(total),
    NOW()
FROM orders
WHERE order_date >= DATE_TRUNC('month', CURRENT_DATE)
GROUP BY DATE_TRUNC('month', order_date)
ON CONFLICT (year_month) DO UPDATE SET
    order_count = EXCLUDED.order_count,
    revenue = EXCLUDED.revenue,
    avg_order_value = EXCLUDED.avg_order_value,
    updated_at = EXCLUDED.updated_at;
```

### 3.5 Adding Redundant Columns for Sorting/Filtering

```sql
-- BEFORE: Sort by category requires join
SELECT p.* FROM products p
JOIN categories c ON p.category_id = c.category_id
ORDER BY c.category_name, p.product_name;

-- AFTER: Category name stored in products
ALTER TABLE products ADD COLUMN category_name VARCHAR(100);

-- Index for fast sorting
CREATE INDEX idx_products_cat_name ON products(category_name, product_name);

-- Fast query
SELECT * FROM products ORDER BY category_name, product_name;
```

---

## 4. Maintaining Denormalized Data

### 4.1 Using Triggers

```sql
-- Trigger to maintain order total
CREATE OR REPLACE FUNCTION update_order_total()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE orders
    SET total = (
        SELECT COALESCE(SUM(quantity * unit_price), 0)
        FROM order_items
        WHERE order_id = COALESCE(NEW.order_id, OLD.order_id)
    )
    WHERE order_id = COALESCE(NEW.order_id, OLD.order_id);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_order_items_total
AFTER INSERT OR UPDATE OR DELETE ON order_items
FOR EACH ROW EXECUTE FUNCTION update_order_total();
```

### 4.2 Using Materialized Views

```sql
-- Materialized view (PostgreSQL)
CREATE MATERIALIZED VIEW mv_customer_stats AS
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count,
    SUM(o.total) AS lifetime_value,
    MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

-- Create index on materialized view
CREATE INDEX idx_mv_customer_stats_value ON mv_customer_stats(lifetime_value DESC);

-- Refresh periodically
REFRESH MATERIALIZED VIEW mv_customer_stats;

-- Refresh concurrently (allows reads during refresh, requires unique index)
CREATE UNIQUE INDEX idx_mv_customer_stats_pk ON mv_customer_stats(customer_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_customer_stats;
```

### 4.3 Application-Level Updates

```python
# Python example: Update denormalized data in application
class OrderService:
    def add_order_item(self, order_id, product_id, quantity, price):
        # Insert order item
        self.db.execute("""
            INSERT INTO order_items (order_id, product_id, quantity, unit_price)
            VALUES (%s, %s, %s, %s)
        """, (order_id, product_id, quantity, price))

        # Update denormalized total
        self.db.execute("""
            UPDATE orders
            SET total = total + %s,
                item_count = item_count + 1,
                updated_at = NOW()
            WHERE order_id = %s
        """, (quantity * price, order_id))

        self.db.commit()
```

### 4.4 Batch/ETL Updates

```sql
-- Scheduled job to refresh denormalized data
-- Run nightly or hourly depending on requirements

-- Full refresh of summary table
TRUNCATE TABLE daily_sales_summary;
INSERT INTO daily_sales_summary
SELECT
    DATE(order_date) AS order_date,
    COUNT(*) AS order_count,
    SUM(total) AS revenue
FROM orders
GROUP BY DATE(order_date);

-- Incremental update
INSERT INTO daily_sales_summary (order_date, order_count, revenue)
SELECT
    DATE(order_date),
    COUNT(*),
    SUM(total)
FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '1 day'
GROUP BY DATE(order_date)
ON CONFLICT (order_date) DO UPDATE SET
    order_count = EXCLUDED.order_count,
    revenue = EXCLUDED.revenue;
```

---

## 5. Denormalization Patterns

### 5.1 Caching Pattern

```sql
-- Cache expensive computation
ALTER TABLE products ADD COLUMN avg_rating DECIMAL(3, 2);
ALTER TABLE products ADD COLUMN review_count INT DEFAULT 0;

-- Update cache when reviews change
CREATE OR REPLACE FUNCTION update_product_rating()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE products SET
        avg_rating = (SELECT AVG(rating) FROM reviews WHERE product_id = NEW.product_id),
        review_count = (SELECT COUNT(*) FROM reviews WHERE product_id = NEW.product_id)
    WHERE product_id = NEW.product_id;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### 5.2 Snapshot Pattern

```sql
-- Store data as it was at a point in time
CREATE TABLE order_snapshots (
    order_id INT,
    customer_id INT,
    customer_name VARCHAR(100),      -- Snapshot of name at order time
    customer_address TEXT,           -- Snapshot of address
    product_details JSONB,           -- Snapshot of all product info
    created_at TIMESTAMP
);

-- Even if customer moves or product changes, order history is accurate
```

### 5.3 Hierarchy Flattening

```sql
-- Flatten a tree structure for faster queries
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    parent_id INT REFERENCES categories(category_id),
    name VARCHAR(100)
);

-- Denormalized version with path
CREATE TABLE categories_denorm (
    category_id INT PRIMARY KEY,
    parent_id INT,
    name VARCHAR(100),
    full_path TEXT,           -- 'Electronics > Computers > Laptops'
    depth INT,                -- Level in hierarchy
    root_id INT,              -- Top-level ancestor
    all_ancestors INT[]       -- Array of ancestor IDs
);

-- Fast queries:
SELECT * FROM categories_denorm WHERE 5 = ANY(all_ancestors);  -- All descendants of 5
SELECT * FROM categories_denorm WHERE depth = 2;                -- All second-level categories
```

---

## 6. Risks and Mitigations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DENORMALIZATION RISKS                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   RISK: DATA INCONSISTENCY                                                  │
│   ─────────────────────────                                                 │
│   Problem: Denormalized data gets out of sync with source                  │
│   Mitigation:                                                               │
│   • Use triggers for real-time sync                                        │
│   • Periodic reconciliation jobs                                           │
│   • Clear ownership of data updates                                        │
│                                                                              │
│   RISK: UPDATE ANOMALIES                                                    │
│   ───────────────────────                                                   │
│   Problem: Changes require multiple updates                                │
│   Mitigation:                                                               │
│   • Encapsulate updates in transactions                                    │
│   • Use triggers or stored procedures                                      │
│   • Accept eventual consistency for some data                              │
│                                                                              │
│   RISK: STORAGE OVERHEAD                                                    │
│   ────────────────────────                                                  │
│   Problem: Duplicate data increases storage                                │
│   Mitigation:                                                               │
│   • Measure actual storage impact                                          │
│   • Consider compression                                                   │
│   • Only denormalize what's needed                                         │
│                                                                              │
│   RISK: COMPLEXITY                                                          │
│   ────────────────                                                          │
│   Problem: More complex to understand and maintain                         │
│   Mitigation:                                                               │
│   • Document denormalization decisions                                     │
│   • Use naming conventions (e.g., _denorm suffix)                         │
│   • Keep canonical source clear                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Decision Framework

```sql
-- Before denormalizing, answer these questions:

-- 1. Is there a measured performance problem?
EXPLAIN ANALYZE SELECT ... -- Get actual query plan and timing

-- 2. Have you tried other optimizations first?
-- Indexes?
CREATE INDEX idx_orders_customer ON orders(customer_id);
-- Query rewriting?
-- Connection pooling?
-- Caching layer?

-- 3. What's the read/write ratio?
SELECT
    (SELECT COUNT(*) FROM pg_stat_user_tables WHERE relname = 'orders') AS reads,
    (SELECT n_tup_ins + n_tup_upd FROM pg_stat_user_tables WHERE relname = 'orders') AS writes;
-- If writes >> reads, don't denormalize

-- 4. How will you maintain consistency?
-- Document the strategy before implementing

-- 5. What's the rollback plan?
-- Can you remove denormalization if it causes problems?
```

---

## 8. Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DENORMALIZATION BEST PRACTICES                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. NORMALIZE FIRST                                                         │
│      • Start with normalized design                                         │
│      • Measure actual performance                                           │
│      • Denormalize only specific bottlenecks                               │
│                                                                              │
│   2. KEEP SOURCE OF TRUTH                                                    │
│      • Normalized tables are the canonical source                          │
│      • Denormalized data is derived/cached                                 │
│      • Never update denormalized data directly                             │
│                                                                              │
│   3. DOCUMENT EVERYTHING                                                     │
│      • Why was this denormalized?                                          │
│      • How is it kept in sync?                                             │
│      • What queries does it optimize?                                      │
│                                                                              │
│   4. AUTOMATE MAINTENANCE                                                    │
│      • Use triggers or materialized views                                  │
│      • Don't rely on manual processes                                      │
│      • Build monitoring for consistency                                    │
│                                                                              │
│   5. CONSIDER ALTERNATIVES                                                   │
│      • Materialized views (database-managed denormalization)               │
│      • Caching layer (Redis, Memcached)                                    │
│      • Read replicas                                                       │
│      • Better indexes                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

| Technique | Use Case | Maintenance |
|-----------|----------|-------------|
| Calculated columns | Avoid repeated calculations | Trigger or application |
| Duplicated columns | Avoid frequent joins | Trigger or ETL |
| Summary tables | Pre-computed aggregates | Scheduled job or trigger |
| Materialized views | Complex query results | REFRESH command |
| Hierarchy flattening | Tree traversal | Trigger on structure change |
| Snapshots | Historical accuracy | At insert time |

**Key Principles:**
1. Normalize first, denormalize with evidence
2. Keep normalized data as source of truth
3. Automate synchronization
4. Document all denormalization decisions
5. Monitor for data inconsistencies
6. Consider alternatives (indexes, caching, views)
