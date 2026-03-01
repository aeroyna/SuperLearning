# JSONB and Document Storage

## Learning Objectives
- Understand JSON vs JSONB differences
- Master JSONB operators and functions
- Index JSON data effectively
- Design hybrid relational-document schemas

---

## 1. JSON vs JSONB

### Key Differences

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JSON vs JSONB Comparison                          │
│                                                                      │
│  Feature              JSON                JSONB                      │
│  ─────────────────────────────────────────────────────────────────  │
│  Storage              Text (exact copy)   Binary (parsed)           │
│  Whitespace           Preserved           Removed                   │
│  Key Order            Preserved           Not preserved             │
│  Duplicate Keys       Allowed             Last value wins           │
│  Input Speed          Faster              Slower (parsing)          │
│  Processing Speed     Slower              Faster                    │
│  Indexing             Not supported       GIN indexes               │
│                                                                      │
│  Recommendation: Almost always use JSONB                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Storage Comparison

```sql
-- JSON stores exact text
SELECT '{"b": 2,  "a": 1}'::json;
-- Result: {"b": 2,  "a": 1}

-- JSONB normalizes and reorders
SELECT '{"b": 2,  "a": 1}'::jsonb;
-- Result: {"a": 1, "b": 2}

-- Duplicate keys in JSON
SELECT '{"a": 1, "a": 2}'::json -> 'a';  -- 2 (last value)

-- JSONB keeps only last
SELECT '{"a": 1, "a": 2}'::jsonb;  -- {"a": 2}
```

---

## 2. Creating JSONB Data

### Table Definition

```sql
-- Products with JSONB attributes
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    price NUMERIC(10,2),
    attributes JSONB DEFAULT '{}',
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Events with JSONB payload
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_type VARCHAR(50),
    payload JSONB NOT NULL,
    occurred_at TIMESTAMP DEFAULT NOW()
);
```

### Inserting JSONB

```sql
-- Insert with JSONB literal
INSERT INTO products (name, category, price, attributes) VALUES
    ('Laptop Pro', 'electronics', 1299.99, '{
        "brand": "TechCorp",
        "specs": {
            "cpu": "Intel i7",
            "ram": 16,
            "storage": "512GB SSD"
        },
        "colors": ["silver", "space gray"],
        "in_stock": true
    }'),
    ('Wireless Mouse', 'electronics', 49.99, '{
        "brand": "ClickMaster",
        "wireless": true,
        "battery_type": "AA",
        "dpi": [800, 1600, 3200]
    }');

-- Insert using jsonb_build_object
INSERT INTO products (name, category, price, attributes) VALUES
    ('Smart Watch', 'electronics', 299.99,
     jsonb_build_object(
         'brand', 'WristTech',
         'features', jsonb_build_array('heart_rate', 'gps', 'notifications'),
         'water_resistant', true
     ));
```

---

## 3. JSONB Operators

### Extraction Operators

```sql
-- -> returns JSONB
SELECT attributes -> 'brand' FROM products;  -- "TechCorp" (with quotes)
SELECT attributes -> 'specs' -> 'cpu' FROM products;  -- "Intel i7"

-- ->> returns TEXT
SELECT attributes ->> 'brand' FROM products;  -- TechCorp (no quotes)
SELECT attributes -> 'specs' ->> 'cpu' FROM products;  -- Intel i7

-- #> path extraction (returns JSONB)
SELECT attributes #> '{specs,cpu}' FROM products;  -- "Intel i7"

-- #>> path extraction (returns TEXT)
SELECT attributes #>> '{specs,cpu}' FROM products;  -- Intel i7

-- Array element access (0-indexed)
SELECT attributes -> 'colors' -> 0 FROM products;  -- "silver"
SELECT attributes -> 'colors' ->> 1 FROM products;  -- space gray
```

### Containment Operators

```sql
-- @> contains
SELECT * FROM products
WHERE attributes @> '{"brand": "TechCorp"}';

-- Check nested containment
SELECT * FROM products
WHERE attributes @> '{"specs": {"ram": 16}}';

-- <@ contained by
SELECT * FROM products
WHERE '{"brand": "TechCorp", "in_stock": true}'::jsonb <@ attributes;

-- ? key exists
SELECT * FROM products
WHERE attributes ? 'wireless';

-- ?| any key exists
SELECT * FROM products
WHERE attributes ?| array['wireless', 'bluetooth'];

-- ?& all keys exist
SELECT * FROM products
WHERE attributes ?& array['brand', 'in_stock'];
```

### Modification Operators

```sql
-- || concatenate/merge
SELECT '{"a": 1}'::jsonb || '{"b": 2}'::jsonb;
-- {"a": 1, "b": 2}

-- Override existing keys
SELECT '{"a": 1, "b": 2}'::jsonb || '{"b": 3}'::jsonb;
-- {"a": 1, "b": 3}

-- - delete key
SELECT '{"a": 1, "b": 2}'::jsonb - 'a';
-- {"b": 2}

-- - delete by path
SELECT '{"a": {"b": 1, "c": 2}}'::jsonb #- '{a,b}';
-- {"a": {"c": 2}}

-- Delete array element by index
SELECT '["a", "b", "c"]'::jsonb - 1;
-- ["a", "c"]
```

---

## 4. JSONB Functions

### Construction Functions

```sql
-- Build object
SELECT jsonb_build_object(
    'name', 'John',
    'age', 30,
    'active', true
);
-- {"age": 30, "name": "John", "active": true}

-- Build array
SELECT jsonb_build_array(1, 'two', true, null);
-- [1, "two", true, null]

-- Convert row to JSONB
SELECT to_jsonb(ROW('John', 30, true));
-- {"f1": "John", "f2": 30, "f3": true}

-- Table row to JSONB
SELECT to_jsonb(p) FROM products p WHERE id = 1;

-- Aggregate to JSONB array
SELECT jsonb_agg(name) FROM products;
-- ["Laptop Pro", "Wireless Mouse", "Smart Watch"]

-- Aggregate to JSONB object
SELECT jsonb_object_agg(name, price) FROM products;
-- {"Laptop Pro": 1299.99, "Wireless Mouse": 49.99, ...}
```

### Inspection Functions

```sql
-- Type of JSONB value
SELECT jsonb_typeof('"hello"'::jsonb);     -- string
SELECT jsonb_typeof('123'::jsonb);          -- number
SELECT jsonb_typeof('true'::jsonb);         -- boolean
SELECT jsonb_typeof('null'::jsonb);         -- null
SELECT jsonb_typeof('[]'::jsonb);           -- array
SELECT jsonb_typeof('{}'::jsonb);           -- object

-- Array length
SELECT jsonb_array_length('["a","b","c"]'::jsonb);  -- 3

-- Get object keys
SELECT jsonb_object_keys('{"a":1, "b":2}'::jsonb);
-- a
-- b

-- Pretty print
SELECT jsonb_pretty('{"a":1,"b":{"c":2}}'::jsonb);
/*
{
    "a": 1,
    "b": {
        "c": 2
    }
}
*/
```

### Iteration Functions

```sql
-- Iterate object key-value pairs
SELECT * FROM jsonb_each('{"a": 1, "b": "two"}'::jsonb);
-- key | value
-- a   | 1
-- b   | "two"

-- With text values
SELECT * FROM jsonb_each_text('{"a": 1, "b": "two"}'::jsonb);
-- key | value
-- a   | 1
-- b   | two

-- Iterate array elements
SELECT * FROM jsonb_array_elements('[1, 2, 3]'::jsonb);
-- value
-- 1
-- 2
-- 3

-- With text
SELECT * FROM jsonb_array_elements_text('["a", "b"]'::jsonb);
-- value
-- a
-- b
```

### Modification Functions

```sql
-- Set value at path
SELECT jsonb_set(
    '{"a": {"b": 1}}'::jsonb,
    '{a,c}',
    '2'::jsonb
);
-- {"a": {"b": 1, "c": 2}}

-- Set with create_if_missing = false (default true)
SELECT jsonb_set(
    '{"a": 1}'::jsonb,
    '{b}',
    '2'::jsonb,
    false
);
-- {"a": 1} (b not created because it doesn't exist)

-- Insert into array
SELECT jsonb_insert(
    '{"a": [1, 2, 3]}'::jsonb,
    '{a, 1}',
    '"new"'::jsonb
);
-- {"a": [1, "new", 2, 3]}

-- Insert after (not before)
SELECT jsonb_insert(
    '["a", "b"]'::jsonb,
    '{1}',
    '"new"'::jsonb,
    true  -- insert after
);
-- ["a", "b", "new"]

-- Strip nulls
SELECT jsonb_strip_nulls('{"a": 1, "b": null}'::jsonb);
-- {"a": 1}
```

---

## 5. Indexing JSONB

### GIN Index

```sql
-- Default GIN index (supports @>, ?, ?&, ?|)
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- Query using index
SELECT * FROM products WHERE attributes @> '{"brand": "TechCorp"}';
SELECT * FROM products WHERE attributes ? 'wireless';

-- GIN with jsonb_path_ops (smaller, only @>)
CREATE INDEX idx_products_attrs_path ON products
USING GIN (attributes jsonb_path_ops);

-- More efficient for containment queries
SELECT * FROM products WHERE attributes @> '{"specs": {"ram": 16}}';
```

### Expression Indexes

```sql
-- Index specific JSONB field
CREATE INDEX idx_products_brand ON products ((attributes ->> 'brand'));

-- Query using expression index
SELECT * FROM products WHERE attributes ->> 'brand' = 'TechCorp';

-- Index nested field
CREATE INDEX idx_products_cpu ON products ((attributes #>> '{specs,cpu}'));

-- Partial index on JSONB condition
CREATE INDEX idx_products_in_stock ON products (id)
WHERE (attributes ->> 'in_stock')::boolean = true;
```

### Index Selection Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                    JSONB Index Selection                             │
│                                                                      │
│  Query Type                      Best Index                          │
│  ─────────────────────────────────────────────────────────────────  │
│  @> containment                  GIN (jsonb_ops or jsonb_path_ops)  │
│  ? key existence                 GIN (jsonb_ops only)               │
│  ->> = 'value'                   B-tree expression index            │
│  ->> LIKE '%pattern%'            GIN trigram on expression          │
│  Complex nested queries          GIN (jsonb_path_ops)               │
│                                                                      │
│  jsonb_ops:      Larger, supports all operators                     │
│  jsonb_path_ops: Smaller, only @> (containment)                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. JSONB Path Queries (PostgreSQL 12+)

### SQL/JSON Path Language

```sql
-- Basic path query
SELECT jsonb_path_query(
    '{"a": [1, 2, 3]}',
    '$.a[*]'
);
-- 1, 2, 3 (as separate rows)

-- Path with filter
SELECT jsonb_path_query(
    '[{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]',
    '$[*] ? (@.age > 26)'
);
-- {"name": "John", "age": 30}

-- Check if path exists
SELECT jsonb_path_exists(
    '{"a": {"b": 1}}',
    '$.a.b'
);
-- true

-- Get first match
SELECT jsonb_path_query_first(
    '[1, 2, 3, 4, 5]',
    '$[*] ? (@ > 3)'
);
-- 4
```

### Path Expressions

```sql
-- Nested access
SELECT jsonb_path_query('{"a":{"b":{"c":1}}}', '$.a.b.c');  -- 1

-- Array access
SELECT jsonb_path_query('[1,2,3]', '$[0]');    -- 1
SELECT jsonb_path_query('[1,2,3]', '$[last]'); -- 3
SELECT jsonb_path_query('[1,2,3]', '$[0 to 1]'); -- 1, 2

-- Wildcard
SELECT jsonb_path_query('{"a":1,"b":2}', '$.*');  -- 1, 2
SELECT jsonb_path_query('[[1,2],[3,4]]', '$[*][*]'); -- 1,2,3,4

-- Filter expressions
SELECT jsonb_path_query(
    '[{"price": 100}, {"price": 200}, {"price": 50}]',
    '$[*] ? (@.price >= 100)'
);
-- {"price": 100}, {"price": 200}

-- Arithmetic in filters
SELECT jsonb_path_query(
    '[{"qty": 10, "price": 5}, {"qty": 5, "price": 20}]',
    '$[*] ? (@.qty * @.price > 50)'
);
```

### Path in WHERE Clause

```sql
-- Using @@ operator for path predicates
SELECT * FROM products
WHERE attributes @@ '$.specs.ram > 8';

-- Using @? for path existence
SELECT * FROM products
WHERE attributes @? '$.colors[*] ? (@ == "silver")';
```

---

## 7. Document Database Patterns

### Hybrid Schema Design

```sql
-- Users with flexible profile data
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    -- Structured data
    created_at TIMESTAMP DEFAULT NOW(),
    -- Flexible profile as JSONB
    profile JSONB DEFAULT '{}'
);

-- Profile might contain:
-- {
--   "name": "John Doe",
--   "preferences": {"theme": "dark", "notifications": true},
--   "social": {"twitter": "@john", "linkedin": "johndoe"},
--   "interests": ["programming", "music"]
-- }

-- Query structured and JSONB together
SELECT
    email,
    profile ->> 'name' AS name,
    profile -> 'preferences' ->> 'theme' AS theme
FROM users
WHERE created_at > '2024-01-01'
  AND profile @> '{"preferences": {"notifications": true}}';
```

### Event Sourcing

```sql
-- Events table with JSONB payload
CREATE TABLE domain_events (
    id BIGSERIAL PRIMARY KEY,
    aggregate_type VARCHAR(50) NOT NULL,
    aggregate_id UUID NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_data JSONB NOT NULL,
    metadata JSONB DEFAULT '{}',
    occurred_at TIMESTAMP DEFAULT NOW(),
    version INTEGER NOT NULL
);

CREATE INDEX idx_events_aggregate ON domain_events (aggregate_type, aggregate_id, version);
CREATE INDEX idx_events_type ON domain_events USING GIN (event_data);

-- Insert event
INSERT INTO domain_events (aggregate_type, aggregate_id, event_type, event_data, version)
VALUES (
    'Order',
    'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',
    'OrderPlaced',
    '{
        "customer_id": 123,
        "items": [
            {"product_id": 1, "quantity": 2, "price": 29.99},
            {"product_id": 5, "quantity": 1, "price": 49.99}
        ],
        "total": 109.97
    }',
    1
);
```

### Configuration Storage

```sql
-- Application settings with JSONB
CREATE TABLE app_settings (
    id SERIAL PRIMARY KEY,
    key VARCHAR(100) UNIQUE NOT NULL,
    value JSONB NOT NULL,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Store complex settings
INSERT INTO app_settings (key, value) VALUES
    ('email', '{
        "smtp_host": "smtp.example.com",
        "smtp_port": 587,
        "use_tls": true,
        "templates": {
            "welcome": "Welcome to our service!",
            "reset": "Click here to reset password"
        }
    }'),
    ('features', '{
        "dark_mode": true,
        "beta_features": ["new_ui", "ai_assist"],
        "rate_limits": {"api": 1000, "uploads": 50}
    }');

-- Get specific setting
SELECT value -> 'templates' ->> 'welcome'
FROM app_settings
WHERE key = 'email';
```

---

## 8. Performance Tips

### Query Optimization

```sql
-- Bad: Extracting in WHERE without index
SELECT * FROM products WHERE attributes ->> 'brand' = 'TechCorp';

-- Good: Create expression index first
CREATE INDEX idx_brand ON products ((attributes ->> 'brand'));

-- Better for containment: Use @> with GIN index
SELECT * FROM products WHERE attributes @> '{"brand": "TechCorp"}';

-- Avoid: Casting JSONB to text for comparison
SELECT * FROM products WHERE (attributes -> 'specs' ->> 'ram')::int > 8;

-- Better: Use JSONB path expressions (PostgreSQL 12+)
SELECT * FROM products WHERE attributes @@ '$.specs.ram > 8';
```

### Schema Design Tips

```sql
-- Keep frequently queried fields as regular columns
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL,       -- Frequently queried
    status VARCHAR(20) NOT NULL,        -- Frequently queried
    total NUMERIC(10,2) NOT NULL,       -- Frequently aggregated
    details JSONB NOT NULL              -- Variable structure
);

-- Index the extracted columns
CREATE INDEX idx_orders_customer ON orders (customer_id);
CREATE INDEX idx_orders_status ON orders (status);

-- GIN for flexible queries on details
CREATE INDEX idx_orders_details ON orders USING GIN (details);
```

---

## Summary

| Operation | Operator/Function | Returns |
|-----------|-------------------|---------|
| Get field | `->` | JSONB |
| Get field as text | `->>` | TEXT |
| Contains | `@>` | BOOLEAN |
| Key exists | `?` | BOOLEAN |
| Merge | `\|\|` | JSONB |
| Delete key | `-` | JSONB |
| Set value | `jsonb_set()` | JSONB |
| Iterate | `jsonb_each()` | SET |

---

## Further Reading

- PostgreSQL JSON Functions documentation
- SQL/JSON Path Language
- "The Art of PostgreSQL" - JSON chapter
