# BRIN and Bloom Indexes

## Learning Objectives
- Understand BRIN (Block Range Index) for large tables
- Use Bloom filters for multi-column queries
- Choose when specialized indexes are appropriate
- Configure and maintain these index types

---

## 1. BRIN (Block Range Index)

### BRIN Concept

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BRIN Index Structure                              │
│                                                                      │
│  Table with naturally ordered data (e.g., timestamp):                │
│                                                                      │
│  Physical Storage:                                                   │
│  ┌─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐      │
│  │ Block 1 │ Block 2 │ Block 3 │ Block 4 │ Block 5 │ Block 6 │      │
│  │ Jan 1-5 │ Jan 6-10│Jan 11-15│Jan 16-20│Jan 21-25│Jan 26-31│      │
│  └─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘      │
│                                                                      │
│  BRIN Index (summary per range of blocks):                          │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ Range │ Blocks  │ Min Value  │ Max Value   │              │       │
│  │   1   │  1-128  │ 2024-01-01 │ 2024-01-15  │              │       │
│  │   2   │ 129-256 │ 2024-01-16 │ 2024-01-31  │              │       │
│  │   3   │ 257-384 │ 2024-02-01 │ 2024-02-14  │              │       │
│  │  ...  │  ...    │    ...     │    ...      │              │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  Query: WHERE date = '2024-01-20'                                    │
│  → Check BRIN: Only Range 2 could contain this date                 │
│  → Scan only blocks 129-256 (not entire table)                       │
│                                                                      │
│  Key: Data must be physically correlated with indexed column!       │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating BRIN Indexes

```sql
-- Time-series data (ideal for BRIN)
CREATE TABLE sensor_readings (
    id BIGSERIAL PRIMARY KEY,
    sensor_id INTEGER,
    reading_time TIMESTAMP NOT NULL,
    value NUMERIC(10,2)
);

-- BRIN index on timestamp
CREATE INDEX idx_readings_time_brin ON sensor_readings
USING BRIN (reading_time);

-- With custom pages per range (default 128)
CREATE INDEX idx_readings_time_brin ON sensor_readings
USING BRIN (reading_time) WITH (pages_per_range = 64);

-- Smaller ranges = more precise, larger index
-- Larger ranges = less precise, smaller index
```

### When BRIN Works Well

```sql
-- Ideal scenarios:
-- 1. Data inserted in order (append-only)
-- 2. Column values correlate with physical location
-- 3. Large tables (millions+ rows)
-- 4. Range/comparison queries

-- Good: Log tables with timestamp
CREATE TABLE logs (
    id BIGSERIAL,
    created_at TIMESTAMP DEFAULT NOW(),
    level VARCHAR(10),
    message TEXT
);
-- Rows inserted chronologically → BRIN on created_at is effective

-- Bad: Randomly updated table
CREATE TABLE users (
    id SERIAL,
    email VARCHAR(255),
    last_login TIMESTAMP
);
-- last_login updates randomly → BRIN won't help
```

### BRIN vs B-Tree Size

```sql
-- Compare index sizes on large table
-- Table: 100 million rows, 10GB

-- B-Tree index: ~2GB
CREATE INDEX idx_btree ON large_table (timestamp_col);

-- BRIN index: ~100KB (20,000x smaller!)
CREATE INDEX idx_brin ON large_table USING BRIN (timestamp_col);

-- Trade-off: BRIN may scan more blocks than B-Tree
-- But for large tables, index size savings often worth it
```

### BRIN Operators

```sql
-- BRIN supports same operators as B-Tree:
-- <, <=, =, >=, >, BETWEEN

SELECT * FROM sensor_readings
WHERE reading_time >= '2024-01-01'
  AND reading_time < '2024-02-01';

-- BRIN eliminates block ranges that can't match
-- Then scans remaining blocks fully

-- Check how much BRIN helps
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM sensor_readings
WHERE reading_time = '2024-06-15';
-- Look at "Rows Removed by Index Recheck"
```

### Multi-Column BRIN

```sql
-- BRIN on multiple columns
CREATE TABLE events (
    id BIGSERIAL,
    event_time TIMESTAMP,
    user_id INTEGER,
    event_type VARCHAR(50)
);

-- If data is ordered by time, then by user_id within each time period
CREATE INDEX idx_events_brin ON events
USING BRIN (event_time, user_id);

-- Effective for queries on both columns:
SELECT * FROM events
WHERE event_time BETWEEN '2024-01-01' AND '2024-01-31'
  AND user_id = 12345;
```

### BRIN Maintenance

```sql
-- BRIN summarizes existing data
-- New data may not be summarized immediately

-- Check unsummarized ranges
SELECT * FROM brin_page_items(get_raw_page('idx_readings_time_brin', 0), 'idx_readings_time_brin');

-- Update BRIN index summaries
SELECT brin_summarize_new_values('idx_readings_time_brin');

-- Or summarize specific range
SELECT brin_summarize_range('idx_readings_time_brin', 1000);

-- Automatic summarization with autosummarize
CREATE INDEX idx_brin ON table USING BRIN (column)
WITH (autosummarize = on);

-- Desummarize (invalidate) a range
SELECT brin_desummarize_range('idx_readings_time_brin', 1000);
```

---

## 2. Bloom Indexes

### Bloom Filter Concept

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Bloom Filter Concept                              │
│                                                                      │
│  Probabilistic data structure for set membership                     │
│                                                                      │
│  Adding elements:                                                    │
│  ┌────────────────────────────────────────────────────────┐         │
│  │  "apple" → hash1("apple")=3, hash2("apple")=7          │         │
│  │  Bit array: [0,0,0,1,0,0,0,1,0,0,0,0,0,0,0,0]          │         │
│  │              positions 3 and 7 set to 1                │         │
│  └────────────────────────────────────────────────────────┘         │
│                                                                      │
│  Checking membership:                                                │
│  ┌────────────────────────────────────────────────────────┐         │
│  │  "apple" → Check bits 3 and 7: both 1 → MAYBE present │         │
│  │  "grape" → Check bits 2 and 5: bit 2 is 0 → NOT present│         │
│  └────────────────────────────────────────────────────────┘         │
│                                                                      │
│  Properties:                                                         │
│  • NO false negatives: If filter says "no", definitely not there    │
│  • POSSIBLE false positives: "Maybe" could be wrong                 │
│  • Very space efficient: ~10 bits per element                       │
│  • Fast lookup: O(k) where k = number of hash functions             │
└─────────────────────────────────────────────────────────────────────┘
```

### PostgreSQL Bloom Index

```sql
-- Enable bloom extension
CREATE EXTENSION bloom;

-- Create bloom index on multiple columns
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    product_id INTEGER,
    warehouse_id INTEGER,
    shipping_method VARCHAR(20),
    status VARCHAR(20),
    order_date DATE
);

-- Bloom index covering many columns
CREATE INDEX idx_orders_bloom ON orders USING BLOOM
    (customer_id, product_id, warehouse_id, shipping_method, status);

-- Queries that benefit from bloom:
SELECT * FROM orders
WHERE customer_id = 1000
  AND product_id = 500
  AND status = 'shipped';
```

### When to Use Bloom

```sql
-- Bloom is good for:
-- 1. Many columns in WHERE clause
-- 2. Random combinations of columns
-- 3. Each column has many distinct values
-- 4. Equality queries only (=)

-- Alternative without Bloom: Multiple B-Tree indexes
CREATE INDEX idx_customer ON orders (customer_id);
CREATE INDEX idx_product ON orders (product_id);
CREATE INDEX idx_warehouse ON orders (warehouse_id);
-- → Many indexes, large total size

-- With Bloom: Single compact index
CREATE INDEX idx_orders_bloom ON orders USING BLOOM
    (customer_id, product_id, warehouse_id);
-- → One index, much smaller

-- Bloom is NOT good for:
-- • Range queries (>, <, BETWEEN)
-- • ORDER BY
-- • Single-column equality (B-Tree better)
-- • Small tables
```

### Bloom Configuration

```sql
-- Default bloom index
CREATE INDEX idx_bloom ON orders USING BLOOM (col1, col2, col3);

-- Customize signature length and bits per column
CREATE INDEX idx_bloom ON orders USING BLOOM (col1, col2, col3)
WITH (
    length = 80,           -- Signature length in bits (default 80)
    col1 = 2,              -- Bits for col1 (default 2)
    col2 = 4,              -- More bits = fewer false positives
    col3 = 2
);

-- More bits per column:
--   • Fewer false positives
--   • Larger index size
--   • Tune based on cardinality
```

### Bloom vs Multiple B-Trees

```sql
-- Comparison scenario: 6-column filter

-- B-Tree approach
CREATE INDEX idx_a ON table (a);
CREATE INDEX idx_b ON table (b);
CREATE INDEX idx_c ON table (c);
CREATE INDEX idx_d ON table (d);
CREATE INDEX idx_e ON table (e);
CREATE INDEX idx_f ON table (f);

-- Query: WHERE a = 1 AND b = 2 AND c = 3 AND d = 4 AND e = 5 AND f = 6
-- Planner picks one index, scans, rechecks others
-- Or uses BitmapAnd (expensive)

-- Bloom approach
CREATE INDEX idx_bloom ON table USING BLOOM (a, b, c, d, e, f);

-- Single index handles all combinations efficiently
-- Trade-off: Some false positives require table rechecks
```

---

## 3. Comparison and Selection

### Index Type Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                 Specialized Index Comparison                         │
│                                                                      │
│  Feature          │ BRIN              │ Bloom              │         │
│  ─────────────────────────────────────────────────────────────────  │
│  Size             │ Very small        │ Small              │         │
│  Build speed      │ Fast              │ Fast               │         │
│  Query type       │ Range & equality  │ Equality only      │         │
│  Columns          │ 1-32              │ 1-32               │         │
│  Requirement      │ Physical ordering │ None               │         │
│  Precision        │ Block ranges      │ Probabilistic      │         │
│  False positives  │ No                │ Yes (configurable) │         │
│  Best for         │ Time-series, logs │ Multi-column AND   │         │
└─────────────────────────────────────────────────────────────────────┘
```

### Decision Guide

```sql
-- Use BRIN when:
-- ✓ Table is very large (GB to TB)
-- ✓ Data has natural physical ordering
-- ✓ Mostly append-only writes
-- ✓ Range queries on ordered column
-- ✓ Want minimal index overhead

-- Use Bloom when:
-- ✓ Queries filter on many columns
-- ✓ Random column combinations
-- ✓ Only equality checks
-- ✓ High cardinality columns
-- ✓ Want single index for multiple columns

-- Use B-Tree when:
-- ✓ General purpose
-- ✓ Single column queries
-- ✓ ORDER BY optimization
-- ✓ Range and equality
-- ✓ Exact results (no false positives)
```

---

## 4. Practical Examples

### Time-Series with BRIN

```sql
-- IoT sensor data
CREATE TABLE iot_data (
    id BIGSERIAL,
    device_id INTEGER,
    recorded_at TIMESTAMPTZ NOT NULL,
    temperature NUMERIC(5,2),
    humidity NUMERIC(5,2),
    pressure NUMERIC(7,2)
) PARTITION BY RANGE (recorded_at);

-- Create partitions for each month
CREATE TABLE iot_data_2024_01 PARTITION OF iot_data
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- BRIN on each partition
CREATE INDEX idx_iot_2024_01_brin ON iot_data_2024_01
USING BRIN (recorded_at) WITH (pages_per_range = 32);

-- Query within time range
EXPLAIN (ANALYZE) SELECT AVG(temperature)
FROM iot_data
WHERE recorded_at BETWEEN '2024-01-15' AND '2024-01-16'
  AND device_id = 100;
```

### Log Analysis with BRIN

```sql
-- Application logs
CREATE TABLE app_logs (
    id BIGSERIAL,
    log_time TIMESTAMP NOT NULL DEFAULT NOW(),
    level VARCHAR(10),
    service VARCHAR(50),
    message TEXT,
    metadata JSONB
);

-- BRIN on timestamp (logs are naturally ordered)
CREATE INDEX idx_logs_time_brin ON app_logs USING BRIN (log_time);

-- B-Tree on level for equality
CREATE INDEX idx_logs_level ON app_logs (level);

-- Query recent errors
SELECT * FROM app_logs
WHERE log_time >= NOW() - INTERVAL '1 hour'
  AND level = 'ERROR';
-- Uses BRIN for time range, B-Tree for level filter
```

### Multi-Column Filter with Bloom

```sql
-- E-commerce product search
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    category_id INTEGER,
    brand_id INTEGER,
    color_id INTEGER,
    size_id INTEGER,
    material_id INTEGER,
    price_range INTEGER,  -- 1-5 buckets
    name VARCHAR(200),
    active BOOLEAN DEFAULT true
);

-- Bloom for common filter combinations
CREATE EXTENSION IF NOT EXISTS bloom;

CREATE INDEX idx_products_bloom ON products USING BLOOM
    (category_id, brand_id, color_id, size_id, material_id, price_range)
WITH (length = 80);

-- Works well for:
SELECT * FROM products
WHERE category_id = 5
  AND color_id = 2
  AND price_range = 3
  AND active = true;

-- Also works for different combinations:
SELECT * FROM products
WHERE brand_id = 10
  AND material_id = 7
  AND active = true;
```

---

## 5. Monitoring and Tuning

### BRIN Effectiveness

```sql
-- Check correlation (how well data is ordered)
SELECT correlation FROM pg_stats
WHERE tablename = 'sensor_readings'
  AND attname = 'reading_time';
-- Close to 1 or -1 = good for BRIN
-- Close to 0 = bad for BRIN

-- Check query plan
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM sensor_readings
WHERE reading_time BETWEEN '2024-06-01' AND '2024-06-02';

-- Look for:
-- • Bitmap Heap Scan
-- • Rows Removed by Index Recheck (should be low)
-- • Buffers (shared hit) vs total table buffers
```

### Bloom Tuning

```sql
-- Test bloom effectiveness
EXPLAIN (ANALYZE)
SELECT * FROM orders
WHERE customer_id = 100
  AND product_id = 500
  AND status = 'shipped';

-- Metrics to watch:
-- • Rows Removed by Index Recheck
-- • If high, increase bits per column

-- Adjust bloom parameters
DROP INDEX idx_orders_bloom;
CREATE INDEX idx_orders_bloom ON orders USING BLOOM
    (customer_id, product_id, status)
WITH (
    length = 96,      -- Larger signature
    customer_id = 4,  -- More bits for high-cardinality
    product_id = 4,
    status = 2        -- Fewer bits for low-cardinality
);
```

---

## Summary

| Index Type | Best Use Case | Size | Requirements |
|------------|---------------|------|--------------|
| BRIN | Time-series, logs | Tiny | Physical ordering |
| Bloom | Multi-column AND | Small | Multiple equality filters |
| B-Tree | General purpose | Medium | Any pattern |
| GIN | Arrays, JSONB, FTS | Large | Containment queries |
| GiST | Spatial, ranges | Medium | Geometric/range ops |

---

## Further Reading

- PostgreSQL BRIN documentation
- Bloom extension documentation
- "PostgreSQL 14 Internals" - Specialized indexes
