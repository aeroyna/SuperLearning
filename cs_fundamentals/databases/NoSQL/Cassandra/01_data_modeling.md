# Data Modeling

## Learning Objectives
- Design tables based on query patterns
- Understand partition and clustering keys
- Apply denormalization strategies
- Avoid common modeling anti-patterns

---

## 1. Query-Driven Design

### The Cassandra Way

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Design Philosophy                                 │
│                                                                      │
│  Relational (SQL):        Cassandra (CQL):                          │
│  ┌───────────────┐        ┌───────────────┐                         │
│  │ Design Schema │        │ Define Queries│                         │
│  │      ↓        │        │      ↓        │                         │
│  │ Write Queries │        │ Design Tables │                         │
│  │      ↓        │        │      ↓        │                         │
│  │ Optimize Later│        │ Denormalize   │                         │
│  └───────────────┘        └───────────────┘                         │
│                                                                      │
│  Key Principles:                                                     │
│  1. Know your queries BEFORE designing tables                       │
│  2. One table per query pattern                                     │
│  3. Denormalization is expected                                     │
│  4. Duplicate data for different access patterns                    │
│  5. No joins - data must be in single partition                    │
└─────────────────────────────────────────────────────────────────────┘
```

### Workflow Example

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Design Workflow                                   │
│                                                                      │
│  Step 1: Define Application Queries                                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Q1: Get user profile by user_id                             │    │
│  │ Q2: Get all orders for a user, sorted by date               │    │
│  │ Q3: Get order details by order_id                           │    │
│  │ Q4: Get all orders in a specific status                     │    │
│  │ Q5: Get daily order totals for reporting                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Step 2: Design Tables for Each Query                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Q1 → users_by_id (user_id PK)                               │    │
│  │ Q2 → orders_by_user (user_id PK, order_date CK DESC)        │    │
│  │ Q3 → orders_by_id (order_id PK)                             │    │
│  │ Q4 → orders_by_status (status PK, order_date CK)            │    │
│  │ Q5 → daily_order_totals (date PK)                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Step 3: Validate Access Patterns                                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ ✓ All queries use partition key                             │    │
│  │ ✓ No full table scans                                       │    │
│  │ ✓ Clustering order matches query needs                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Primary Key Design

### Primary Key Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Primary Key Structure                             │
│                                                                      │
│  PRIMARY KEY ((partition_key), clustering_key1, clustering_key2)    │
│               └──────┬──────┘  └─────────────┬─────────────────┘    │
│                      │                        │                      │
│              Data Location              Sort Order                   │
│                                                                      │
│  Examples:                                                           │
│                                                                      │
│  -- Simple partition key                                            │
│  PRIMARY KEY (user_id)                                              │
│                                                                      │
│  -- Partition key + clustering key                                  │
│  PRIMARY KEY (user_id, created_at)                                  │
│                                                                      │
│  -- Composite partition key                                         │
│  PRIMARY KEY ((country, city), timestamp)                           │
│                                                                      │
│  -- Multiple clustering keys                                        │
│  PRIMARY KEY (sensor_id, year, month, day)                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Partition Key

```sql
-- Partition key determines which node stores the data
-- Hash(partition_key) → token → node

-- Good: High cardinality, evenly distributed
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);

-- Composite partition key: Groups related data
CREATE TABLE sensor_data (
    sensor_id TEXT,
    date DATE,
    timestamp TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((sensor_id, date), timestamp)
);
-- All readings for sensor+date on same node
```

### Clustering Key

```sql
-- Clustering key determines sort order within partition
-- Data is stored sorted on disk

CREATE TABLE user_events (
    user_id UUID,
    event_time TIMESTAMP,
    event_type TEXT,
    data TEXT,
    PRIMARY KEY (user_id, event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
-- Most recent events first

-- Multiple clustering keys
CREATE TABLE messages (
    chat_id UUID,
    bucket INT,           -- Time bucket for partition sizing
    message_time TIMESTAMP,
    message_id TIMEUUID,
    content TEXT,
    PRIMARY KEY ((chat_id, bucket), message_time, message_id)
) WITH CLUSTERING ORDER BY (message_time DESC, message_id DESC);
```

---

## 3. Data Types

### Common Types

```sql
-- Scalar types
TEXT, VARCHAR          -- UTF-8 strings
INT, BIGINT, SMALLINT  -- Integers
FLOAT, DOUBLE          -- Floating point
DECIMAL               -- Arbitrary precision
BOOLEAN               -- true/false
TIMESTAMP             -- Date and time
DATE                  -- Date only
TIME                  -- Time only
UUID                  -- Random UUID
TIMEUUID              -- Time-based UUID (sortable)
BLOB                  -- Binary data
INET                  -- IP address

-- Collection types
LIST<type>            -- Ordered, allows duplicates
SET<type>             -- Unordered, unique
MAP<key, value>       -- Key-value pairs

-- Example usage
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    name TEXT,
    tags SET<TEXT>,
    preferences MAP<TEXT, TEXT>,
    login_history LIST<TIMESTAMP>
);
```

### User-Defined Types (UDTs)

```sql
-- Create UDT
CREATE TYPE address (
    street TEXT,
    city TEXT,
    state TEXT,
    zip TEXT,
    country TEXT
);

-- Use UDT in table
CREATE TABLE customers (
    customer_id UUID PRIMARY KEY,
    name TEXT,
    billing_address FROZEN<address>,
    shipping_addresses LIST<FROZEN<address>>
);

-- Insert with UDT
INSERT INTO customers (customer_id, name, billing_address)
VALUES (
    uuid(),
    'John Doe',
    {street: '123 Main St', city: 'NYC', state: 'NY', zip: '10001', country: 'USA'}
);

-- FROZEN: Required for nested collections/UDTs
-- Means entire value must be replaced (no partial updates)
```

### Counter Type

```sql
-- Special type for distributed counters
CREATE TABLE page_views (
    page_url TEXT PRIMARY KEY,
    view_count COUNTER
);

-- Only UPDATE works with counters
UPDATE page_views SET view_count = view_count + 1 WHERE page_url = '/home';

-- Limitations:
-- • No INSERT (only UPDATE)
-- • No mixing with regular columns
-- • Eventual consistency (may have drift)
```

---

## 4. Table Design Patterns

### Time-Series Data

```sql
-- Pattern: Bucket by time period to control partition size
CREATE TABLE metrics (
    sensor_id TEXT,
    date DATE,
    timestamp TIMESTAMP,
    metric_name TEXT,
    value DOUBLE,
    PRIMARY KEY ((sensor_id, date), timestamp, metric_name)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Query recent data
SELECT * FROM metrics
WHERE sensor_id = 'sensor1'
AND date = '2024-01-15'
AND timestamp > '2024-01-15 10:00:00';

-- Daily rollups in separate table
CREATE TABLE daily_metrics (
    sensor_id TEXT,
    date DATE,
    metric_name TEXT,
    min_value DOUBLE,
    max_value DOUBLE,
    avg_value DOUBLE,
    count BIGINT,
    PRIMARY KEY (sensor_id, date, metric_name)
);
```

### User Activity

```sql
-- User timeline
CREATE TABLE user_timeline (
    user_id UUID,
    activity_time TIMESTAMP,
    activity_id TIMEUUID,
    activity_type TEXT,
    details MAP<TEXT, TEXT>,
    PRIMARY KEY (user_id, activity_time, activity_id)
) WITH CLUSTERING ORDER BY (activity_time DESC, activity_id DESC);

-- Get recent activity
SELECT * FROM user_timeline
WHERE user_id = ?
LIMIT 20;
```

### One-to-Many Relationships

```sql
-- Orders for a customer
CREATE TABLE orders_by_customer (
    customer_id UUID,
    order_date DATE,
    order_id UUID,
    total DECIMAL,
    status TEXT,
    items LIST<FROZEN<order_item>>,
    PRIMARY KEY (customer_id, order_date, order_id)
) WITH CLUSTERING ORDER BY (order_date DESC, order_id DESC);

-- Order lookup by ID (denormalized)
CREATE TABLE orders_by_id (
    order_id UUID PRIMARY KEY,
    customer_id UUID,
    order_date DATE,
    total DECIMAL,
    status TEXT,
    items LIST<FROZEN<order_item>>
);
```

### Many-to-Many Relationships

```sql
-- Users and groups (both directions)
CREATE TABLE users_by_group (
    group_id UUID,
    user_id UUID,
    joined_at TIMESTAMP,
    role TEXT,
    PRIMARY KEY (group_id, user_id)
);

CREATE TABLE groups_by_user (
    user_id UUID,
    group_id UUID,
    joined_at TIMESTAMP,
    role TEXT,
    PRIMARY KEY (user_id, group_id)
);

-- Keep both in sync on writes
```

---

## 5. Denormalization Strategies

### Duplicate for Different Queries

```sql
-- Same order data, different access patterns

-- By customer (for customer order history)
CREATE TABLE orders_by_customer (
    customer_id UUID,
    order_date TIMESTAMP,
    order_id UUID,
    -- ... order details
    PRIMARY KEY (customer_id, order_date, order_id)
);

-- By status (for order processing)
CREATE TABLE orders_by_status (
    status TEXT,
    order_date TIMESTAMP,
    order_id UUID,
    customer_id UUID,
    -- ... order details
    PRIMARY KEY ((status), order_date, order_id)
);

-- By date (for daily reports)
CREATE TABLE orders_by_date (
    order_date DATE,
    order_id UUID,
    customer_id UUID,
    status TEXT,
    total DECIMAL,
    PRIMARY KEY (order_date, order_id)
);
```

### Materialized Views (Use with Caution)

```sql
-- Base table
CREATE TABLE orders (
    order_id UUID PRIMARY KEY,
    customer_id UUID,
    order_date TIMESTAMP,
    status TEXT,
    total DECIMAL
);

-- Materialized view for different query pattern
CREATE MATERIALIZED VIEW orders_by_customer AS
    SELECT * FROM orders
    WHERE customer_id IS NOT NULL AND order_id IS NOT NULL
    PRIMARY KEY (customer_id, order_id);

-- Caveats:
-- • Performance overhead on writes
-- • Eventual consistency with base table
-- • Limited query flexibility
-- • Generally prefer manual denormalization
```

---

## 6. Anti-Patterns

### Unbounded Partitions

```sql
-- BAD: All user messages in one partition
CREATE TABLE messages_bad (
    user_id UUID,
    message_time TIMESTAMP,
    message TEXT,
    PRIMARY KEY (user_id, message_time)
);
-- Partition grows forever!

-- GOOD: Bucket by time
CREATE TABLE messages_good (
    user_id UUID,
    month TEXT,            -- '2024-01'
    message_time TIMESTAMP,
    message TEXT,
    PRIMARY KEY ((user_id, month), message_time)
);
-- Query: user_id + current month
-- Archive: delete old month partitions
```

### High-Cardinality Clustering Keys

```sql
-- BAD: UUID as first clustering key
CREATE TABLE events_bad (
    date DATE,
    event_id UUID,
    user_id UUID,
    PRIMARY KEY (date, event_id, user_id)
);
-- Can't efficiently query by user_id!

-- GOOD: Query-friendly clustering order
CREATE TABLE events_good (
    date DATE,
    user_id UUID,
    event_id UUID,
    PRIMARY KEY (date, user_id, event_id)
);
-- Can query: date + user_id
```

### Using Secondary Indexes Heavily

```sql
-- BAD: Relying on secondary indexes
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    email TEXT,
    country TEXT
);
CREATE INDEX ON users(email);
CREATE INDEX ON users(country);
-- Secondary indexes = scatter-gather queries!

-- GOOD: Denormalized lookup tables
CREATE TABLE users_by_email (
    email TEXT PRIMARY KEY,
    user_id UUID
);
-- Direct partition lookup
```

### Collections as Query Filters

```sql
-- BAD: Filtering on collection elements
CREATE TABLE products (
    product_id UUID PRIMARY KEY,
    name TEXT,
    tags SET<TEXT>
);
SELECT * FROM products WHERE tags CONTAINS 'electronics';
-- Full table scan!

-- GOOD: Inverted index table
CREATE TABLE products_by_tag (
    tag TEXT,
    product_id UUID,
    product_name TEXT,
    PRIMARY KEY (tag, product_id)
);
SELECT * FROM products_by_tag WHERE tag = 'electronics';
-- Direct partition lookup
```

---

## 7. Sizing Partitions

### Guidelines

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Partition Sizing                                  │
│                                                                      │
│  Target Size:                                                        │
│  • < 100MB per partition (soft limit)                               │
│  • < 100,000 rows per partition (guideline)                         │
│  • Large partitions = slow queries, compaction issues               │
│                                                                      │
│  Calculate Partition Size:                                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Size = Σ(column_size) × rows + overhead                     │    │
│  │                                                              │    │
│  │ Example: IoT sensor, 1 reading/second                       │    │
│  │ • Row size: ~100 bytes                                      │    │
│  │ • Rows/day: 86,400                                          │    │
│  │ • Partition (1 day): ~8.6 MB ✓                              │    │
│  │                                                              │    │
│  │ • Rows/month: 2,592,000                                     │    │
│  │ • Partition (1 month): ~259 MB ✗ Too large                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Bucketing Strategies:                                               │
│  • Time: day, week, month                                           │
│  • Hash: user_id % 10                                               │
│  • Composite: (user_id, month)                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Bucket Example

```sql
-- Time bucket helper
CREATE TABLE time_series_data (
    entity_id TEXT,
    bucket INT,            -- Calculated: timestamp / bucket_size
    timestamp TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((entity_id, bucket), timestamp)
);

-- Application calculates bucket
-- bucket = unix_timestamp // (24 * 60 * 60)  -- Daily buckets
-- bucket = unix_timestamp // (7 * 24 * 60 * 60)  -- Weekly buckets
```

---

## Summary

| Concept | Key Point |
|---------|-----------|
| Query-first | Design tables based on queries, not entities |
| Partition key | Determines data distribution |
| Clustering key | Determines sort order within partition |
| Denormalization | Duplicate data for different access patterns |
| Partition size | Keep < 100MB, < 100K rows |
| No joins | All data for query in one partition |

---

## Best Practices

```
Design:
✓ Start with query patterns
✓ One table per query
✓ Use composite keys effectively
✓ Plan for data growth

Partition Keys:
✓ High cardinality
✓ Even distribution
✓ Include in all queries

Clustering Keys:
✓ Match query ORDER BY
✓ Consider range queries
✓ Put selective columns first

Avoid:
✗ Unbounded partitions
✗ Secondary indexes for high-cardinality
✗ Relational thinking (joins, normalization)
✗ Large collections in rows
```
