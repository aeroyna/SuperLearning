# Hash, GIN, and GiST Indexes

## Learning Objectives
- Understand Hash index use cases
- Master GIN for full-text and array indexing
- Use GiST for geometric and range data
- Choose the right index type for your data

---

## 1. Hash Indexes

### Hash Index Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Hash Index Structure                              │
│                                                                      │
│  Key → hash(Key) → Bucket Number → Bucket Page                      │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────┐      │
│  │                     Bucket Array                           │      │
│  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐                       │      │
│  │  │ B0 │ │ B1 │ │ B2 │ │ B3 │ │... │                       │      │
│  │  └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └────┘                       │      │
│  └─────│─────│─────│─────│────────────────────────────────────┘      │
│        │     │     │     │                                           │
│        ▼     ▼     ▼     ▼                                           │
│     ┌─────┐   ┌─────┐                                                │
│     │Items│   │Items│  Bucket Pages                                  │
│     │(k,t)│   │(k,t)│  (key, TID pairs)                              │
│     └──┬──┘   └─────┘                                                │
│        │                                                             │
│        ▼                                                             │
│     ┌─────┐                                                          │
│     │Over │  Overflow Pages                                          │
│     │flow │  (when bucket full)                                      │
│     └─────┘                                                          │
│                                                                      │
│  Lookup: O(1) average case                                           │
│  No ordering: Cannot do range queries                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating Hash Indexes

```sql
-- Create hash index
CREATE INDEX idx_users_email_hash ON users USING hash (email);

-- Hash index is ONLY useful for equality
SELECT * FROM users WHERE email = 'john@example.com';  -- ✓ Uses index

-- These CANNOT use hash index:
SELECT * FROM users WHERE email > 'j';     -- ✗
SELECT * FROM users WHERE email LIKE 'j%'; -- ✗
SELECT * FROM users ORDER BY email;        -- ✗
```

### When to Use Hash

```sql
-- Hash vs B-Tree comparison
-- Hash pros:
--   • Smaller index size for long keys
--   • Faster equality lookups (O(1) vs O(log n))
--   • WAL-logged since PostgreSQL 10

-- Hash cons:
--   • Only equality (=) operator
--   • No range, ORDER BY, LIKE support
--   • Rare practical advantage over B-Tree

-- Good use case: Long text keys with only equality checks
CREATE TABLE sessions (
    session_token VARCHAR(256) PRIMARY KEY,
    user_id INTEGER,
    data JSONB
);

CREATE INDEX idx_sessions_token_hash ON sessions USING hash (session_token);

-- B-Tree usually preferred due to versatility
```

---

## 2. GIN (Generalized Inverted Index)

### GIN Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GIN (Inverted Index) Structure                    │
│                                                                      │
│  Document → Extract Keys → Build Inverted Index                      │
│                                                                      │
│  Documents:                                                          │
│  Doc1: "PostgreSQL is a database"                                    │
│  Doc2: "MySQL is also a database"                                    │
│  Doc3: "PostgreSQL has full-text search"                             │
│                                                                      │
│  Inverted Index:                                                     │
│  ┌──────────────┬─────────────────────┐                              │
│  │    Key       │    Posting List     │                              │
│  ├──────────────┼─────────────────────┤                              │
│  │ "database"   │ [Doc1, Doc2]        │                              │
│  │ "full-text"  │ [Doc3]              │                              │
│  │ "mysql"      │ [Doc2]              │                              │
│  │ "postgresql" │ [Doc1, Doc3]        │                              │
│  │ "search"     │ [Doc3]              │                              │
│  └──────────────┴─────────────────────┘                              │
│                                                                      │
│  Query: "postgresql" & "database"                                    │
│  → Intersect posting lists: [Doc1, Doc3] ∩ [Doc1, Doc2] = [Doc1]    │
└─────────────────────────────────────────────────────────────────────┘
```

### GIN for Full-Text Search

```sql
-- Full-text search with GIN
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    body TEXT,
    search_vector TSVECTOR GENERATED ALWAYS AS (
        to_tsvector('english', coalesce(title, '') || ' ' || coalesce(body, ''))
    ) STORED
);

-- Create GIN index on tsvector
CREATE INDEX idx_articles_search ON articles USING GIN (search_vector);

-- Query uses GIN index
SELECT * FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql & performance');

-- Explain shows Bitmap Index Scan on GIN
EXPLAIN SELECT * FROM articles WHERE search_vector @@ to_tsquery('database');
```

### GIN for Arrays

```sql
-- Array containment queries
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[]
);

CREATE INDEX idx_products_tags ON products USING GIN (tags);

-- Containment operator (@>)
SELECT * FROM products WHERE tags @> ARRAY['electronics', 'sale'];

-- Overlap operator (&&)
SELECT * FROM products WHERE tags && ARRAY['electronics', 'computers'];

-- Element existence (ANY)
SELECT * FROM products WHERE 'electronics' = ANY(tags);

-- All these use the GIN index efficiently
```

### GIN for JSONB

```sql
-- JSONB indexing
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB
);

-- Default GIN operator class
CREATE INDEX idx_events_data ON events USING GIN (data);

-- Supports @>, ?, ?|, ?&
SELECT * FROM events WHERE data @> '{"type": "click"}';
SELECT * FROM events WHERE data ? 'user_id';

-- jsonb_path_ops (smaller, only @>)
CREATE INDEX idx_events_data_path ON events USING GIN (data jsonb_path_ops);

-- Only containment with jsonb_path_ops
SELECT * FROM events WHERE data @> '{"user": {"country": "US"}}';
```

### GIN Options

```sql
-- Pending list for faster inserts
CREATE INDEX idx_gin ON table USING GIN (column)
WITH (fastupdate = on, gin_pending_list_limit = 4096);

-- fastupdate = on (default):
--   • Faster inserts (buffered in pending list)
--   • Slightly slower first query after inserts
--   • Pending list flushed on VACUUM or when full

-- fastupdate = off:
--   • Slower inserts
--   • Consistent query performance
--   • Better for read-heavy workloads

-- View pending entries
SELECT * FROM gin_pending_list_stats('idx_gin');
```

---

## 3. GiST (Generalized Search Tree)

### GiST Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GiST Structure                                    │
│                                                                      │
│  Balanced tree with "bounding" predicates at each level              │
│                                                                      │
│  Example: 2D Points                                                  │
│                                                                      │
│           ┌─────────────────────────────────┐                        │
│           │   Root: Bounding Box [0,0-100,100]                       │
│           └─────────────┬───────────────────┘                        │
│               ┌─────────┴─────────┐                                  │
│               ▼                   ▼                                  │
│      ┌────────────────┐  ┌────────────────┐                          │
│      │ [0,0-50,50]    │  │ [50,0-100,100] │  Child bounding boxes   │
│      └───────┬────────┘  └───────┬────────┘                          │
│              │                   │                                   │
│              ▼                   ▼                                   │
│      ┌─────────────┐    ┌─────────────┐                              │
│      │ Points in   │    │ Points in   │    Leaf pages               │
│      │ this region │    │ this region │                              │
│      └─────────────┘    └─────────────┘                              │
│                                                                      │
│  Spatial query: "Find points near (25, 25)"                         │
│  → Only search regions that could contain nearby points              │
└─────────────────────────────────────────────────────────────────────┘
```

### GiST for Geometric Data

```sql
-- Geometric types
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    position POINT
);

CREATE INDEX idx_locations_pos ON locations USING GIST (position);

-- Nearest neighbor search
SELECT name, position <-> point(40.7128, -74.0060) AS distance
FROM locations
ORDER BY position <-> point(40.7128, -74.0060)
LIMIT 10;

-- Contains/within queries
CREATE TABLE regions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    area BOX
);

CREATE INDEX idx_regions_area ON regions USING GIST (area);

SELECT * FROM regions WHERE area @> point(50, 50);  -- Contains point
```

### GiST for Range Types

```sql
-- Range exclusion constraints
CREATE TABLE reservations (
    id SERIAL PRIMARY KEY,
    room_id INTEGER,
    during TSTZRANGE,
    EXCLUDE USING GIST (room_id WITH =, during WITH &&)
);

-- Index already created by EXCLUDE constraint
-- Prevents overlapping reservations

INSERT INTO reservations (room_id, during) VALUES
    (1, '[2024-01-01 14:00, 2024-01-03 11:00)');

-- This fails (overlapping):
INSERT INTO reservations (room_id, during) VALUES
    (1, '[2024-01-02 14:00, 2024-01-04 11:00)');
-- ERROR: conflicting key value violates exclusion constraint

-- Query overlapping ranges
SELECT * FROM reservations
WHERE during && '[2024-01-02, 2024-01-05)'::tstzrange;
```

### GiST for Full-Text (Alternative to GIN)

```sql
-- GiST for tsvector
CREATE INDEX idx_articles_search_gist ON articles USING GIST (search_vector);

-- GIN vs GiST for full-text:
-- ┌─────────────────┬──────────────────┬──────────────────┐
-- │ Characteristic  │ GIN              │ GiST             │
-- ├─────────────────┼──────────────────┼──────────────────┤
-- │ Index size      │ Larger           │ Smaller          │
-- │ Build time      │ Slower           │ Faster           │
-- │ Update speed    │ Slower           │ Faster           │
-- │ Search speed    │ Faster           │ Slower           │
-- │ Exact matches   │ Yes              │ Lossy (recheck)  │
-- └─────────────────┴──────────────────┴──────────────────┘

-- GiST better for:
--   • Frequently updated data
--   • Combined with other GiST-able types

-- GIN better for:
--   • Read-heavy workloads
--   • Large documents
```

---

## 4. Operator Classes

### Understanding Operator Classes

```sql
-- List available operator classes
SELECT am.amname AS index_type,
       opc.opcname AS operator_class,
       opc.opcdefault AS is_default
FROM pg_opclass opc
JOIN pg_am am ON opc.opcmethod = am.oid
WHERE am.amname IN ('gin', 'gist', 'hash')
ORDER BY am.amname, opc.opcname;

-- Common operator classes:

-- GIN:
-- array_ops: Array containment (@>, &&, etc.)
-- jsonb_ops: JSONB all operators
-- jsonb_path_ops: JSONB containment only (@>)
-- tsvector_ops: Full-text search

-- GiST:
-- box_ops: Geometric boxes
-- circle_ops: Circles
-- point_ops: Points
-- range_ops: Range types
-- tsvector_ops: Full-text search
```

### Choosing Operator Class

```sql
-- JSONB: jsonb_ops vs jsonb_path_ops
-- jsonb_ops (default)
CREATE INDEX idx1 ON events USING GIN (data);
-- Supports: @>, ?, ?|, ?&

-- jsonb_path_ops (smaller index)
CREATE INDEX idx2 ON events USING GIN (data jsonb_path_ops);
-- Only supports: @>

-- When to use jsonb_path_ops:
-- • Only need containment queries
-- • Want smaller index size (2-3x smaller)
-- • Don't need key existence checks (?)
```

---

## 5. Comparison and Selection

### Index Type Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Index Selection Guide                             │
│                                                                      │
│  Data Type / Query Pattern          │ Recommended Index             │
│  ─────────────────────────────────────────────────────────────────  │
│  Equality on scalar                  │ B-tree (or Hash)              │
│  Range queries                       │ B-tree                        │
│  ORDER BY                            │ B-tree                        │
│  LIKE 'prefix%'                      │ B-tree (text_pattern_ops)     │
│  LIKE '%pattern%'                    │ GIN (pg_trgm)                 │
│                                      │                               │
│  Array containment (@>, &&)          │ GIN                           │
│  JSONB containment (@>)              │ GIN                           │
│  JSONB key existence (?)             │ GIN (jsonb_ops only)          │
│  Full-text search (@@)               │ GIN (preferred) or GiST       │
│                                      │                               │
│  Range overlap (&&)                  │ GiST                          │
│  Geometric (distance, contains)      │ GiST                          │
│  Exclusion constraints               │ GiST                          │
│  Nearest neighbor (<->)              │ GiST                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Performance Comparison

```sql
-- Compare index sizes
SELECT
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) AS size
FROM pg_indexes
WHERE tablename = 'my_table';

-- Compare query performance
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM articles WHERE search_vector @@ to_tsquery('database');

-- Key metrics:
-- • Execution time
-- • Buffers (shared hit, read)
-- • Rows returned vs estimated
```

---

## 6. Practical Examples

### Multi-Type Search

```sql
-- Combine text search with filters
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    description TEXT,
    price NUMERIC(10,2),
    tags TEXT[],
    attributes JSONB,
    search_vector TSVECTOR GENERATED ALWAYS AS (
        to_tsvector('english', name || ' ' || coalesce(description, ''))
    ) STORED
);

-- Multiple specialized indexes
CREATE INDEX idx_products_search ON products USING GIN (search_vector);
CREATE INDEX idx_products_tags ON products USING GIN (tags);
CREATE INDEX idx_products_attrs ON products USING GIN (attributes jsonb_path_ops);
CREATE INDEX idx_products_price ON products (price);

-- Complex query using multiple indexes
SELECT * FROM products
WHERE search_vector @@ to_tsquery('laptop')
  AND tags @> ARRAY['electronics']
  AND attributes @> '{"brand": "Apple"}'
  AND price < 2000;
```

### Geospatial with PostGIS

```sql
-- PostGIS uses GiST internally
CREATE EXTENSION postgis;

CREATE TABLE places (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    location GEOGRAPHY(POINT, 4326)
);

CREATE INDEX idx_places_location ON places USING GIST (location);

-- Find places within 10km of a point
SELECT name, ST_Distance(location, ST_MakePoint(-73.9857, 40.7484)::geography) AS distance
FROM places
WHERE ST_DWithin(location, ST_MakePoint(-73.9857, 40.7484)::geography, 10000)
ORDER BY distance;
```

---

## Summary

| Index | Best For | Operators |
|-------|----------|-----------|
| Hash | Equality only | = |
| GIN | Multi-value, text | @>, &&, ?, @@ |
| GiST | Spatial, ranges | <->, &&, @> |

---

## Further Reading

- PostgreSQL GIN Index documentation
- PostgreSQL GiST Index documentation
- "PostgreSQL 14 Internals" - Index chapters
