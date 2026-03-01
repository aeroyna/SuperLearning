# CQL and Operations

## Learning Objectives
- Master Cassandra Query Language (CQL) syntax
- Perform CRUD operations effectively
- Use batch statements and lightweight transactions
- Understand query restrictions and workarounds

---

## 1. CQL Fundamentals

### Keyspace Management

```sql
-- Create keyspace
CREATE KEYSPACE IF NOT EXISTS my_app
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};

-- NetworkTopologyStrategy for production
CREATE KEYSPACE my_production_app
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3,
    'dc2': 2
}
AND durable_writes = true;

-- Use keyspace
USE my_app;

-- Alter keyspace
ALTER KEYSPACE my_app
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3
};

-- Drop keyspace
DROP KEYSPACE IF EXISTS old_keyspace;

-- Describe keyspace
DESCRIBE KEYSPACE my_app;
```

### Table Management

```sql
-- Create table
CREATE TABLE users (
    user_id UUID,
    username TEXT,
    email TEXT,
    created_at TIMESTAMP,
    is_active BOOLEAN,
    profile MAP<TEXT, TEXT>,
    PRIMARY KEY (user_id)
);

-- Create table with clustering
CREATE TABLE user_events (
    user_id UUID,
    event_time TIMESTAMP,
    event_type TEXT,
    data TEXT,
    PRIMARY KEY (user_id, event_time)
) WITH CLUSTERING ORDER BY (event_time DESC)
  AND comment = 'User activity events'
  AND gc_grace_seconds = 864000;

-- Alter table
ALTER TABLE users ADD phone TEXT;
ALTER TABLE users DROP phone;
ALTER TABLE users RENAME username TO user_name;

-- Drop table
DROP TABLE IF EXISTS old_table;

-- Truncate table (delete all data)
TRUNCATE users;

-- Describe table
DESCRIBE TABLE users;
```

---

## 2. CRUD Operations

### INSERT

```sql
-- Basic insert
INSERT INTO users (user_id, username, email, created_at)
VALUES (uuid(), 'john_doe', 'john@example.com', toTimestamp(now()));

-- Insert with TTL (Time To Live)
INSERT INTO sessions (session_id, user_id, data)
VALUES (uuid(), ?, ?)
USING TTL 3600;  -- Expires in 1 hour

-- Insert with timestamp
INSERT INTO users (user_id, username, email)
VALUES (uuid(), 'jane', 'jane@example.com')
USING TIMESTAMP 1705344000000000;  -- Microseconds

-- Insert if not exists
INSERT INTO users (user_id, username, email)
VALUES (uuid(), 'unique_user', 'unique@example.com')
IF NOT EXISTS;

-- Insert JSON
INSERT INTO users JSON '{"user_id": "550e8400-e29b-41d4-a716-446655440000", "username": "json_user"}';
```

### SELECT

```sql
-- Basic select
SELECT * FROM users WHERE user_id = ?;

-- Select specific columns
SELECT username, email FROM users WHERE user_id = ?;

-- Select with clustering key range
SELECT * FROM user_events
WHERE user_id = ?
AND event_time >= '2024-01-01'
AND event_time < '2024-02-01';

-- Select with ORDER BY (must match clustering order)
SELECT * FROM user_events
WHERE user_id = ?
ORDER BY event_time DESC
LIMIT 10;

-- Select with ALLOW FILTERING (use sparingly!)
SELECT * FROM users
WHERE email = 'john@example.com'
ALLOW FILTERING;

-- Select distinct partition keys
SELECT DISTINCT user_id FROM user_events;

-- Select count
SELECT COUNT(*) FROM user_events WHERE user_id = ?;

-- Select as JSON
SELECT JSON * FROM users WHERE user_id = ?;

-- Select with functions
SELECT user_id, TTL(username), WRITETIME(email)
FROM users WHERE user_id = ?;

-- Token-based pagination
SELECT * FROM users
WHERE TOKEN(user_id) > TOKEN(?)
LIMIT 100;
```

### UPDATE

```sql
-- Basic update
UPDATE users
SET email = 'newemail@example.com'
WHERE user_id = ?;

-- Update with TTL
UPDATE users USING TTL 86400
SET session_token = 'abc123'
WHERE user_id = ?;

-- Update collections
-- Add to set
UPDATE users SET tags = tags + {'premium'} WHERE user_id = ?;
-- Remove from set
UPDATE users SET tags = tags - {'trial'} WHERE user_id = ?;

-- Add to list
UPDATE users SET history = history + ['action1'] WHERE user_id = ?;
-- Prepend to list
UPDATE users SET history = ['action0'] + history WHERE user_id = ?;
-- Update by index
UPDATE users SET history[0] = 'updated' WHERE user_id = ?;

-- Update map
UPDATE users SET settings['theme'] = 'dark' WHERE user_id = ?;
UPDATE users SET settings = settings + {'locale': 'en'} WHERE user_id = ?;
UPDATE users SET settings = settings - {'deprecated_key'} WHERE user_id = ?;

-- Conditional update (LWT)
UPDATE users
SET email = 'new@example.com'
WHERE user_id = ?
IF email = 'old@example.com';
```

### DELETE

```sql
-- Delete row
DELETE FROM users WHERE user_id = ?;

-- Delete specific column
DELETE email FROM users WHERE user_id = ?;

-- Delete from collection
DELETE tags['premium'] FROM users WHERE user_id = ?;
DELETE settings['old_key'] FROM users WHERE user_id = ?;

-- Delete range (clustering)
DELETE FROM user_events
WHERE user_id = ?
AND event_time < '2024-01-01';

-- Conditional delete
DELETE FROM users
WHERE user_id = ?
IF EXISTS;

DELETE FROM users
WHERE user_id = ?
IF email = 'specific@example.com';
```

---

## 3. Query Restrictions

### What You CAN Query

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CQL Query Rules                                   │
│                                                                      │
│  PRIMARY KEY ((partition_key), clustering1, clustering2)            │
│                                                                      │
│  ✓ Allowed:                                                          │
│  • Partition key equality (required!)                               │
│  • Clustering key equality or range                                 │
│  • Must follow clustering key order (left to right)                 │
│                                                                      │
│  Examples for PRIMARY KEY ((user_id), year, month, day):            │
│                                                                      │
│  ✓ WHERE user_id = ?                                                 │
│  ✓ WHERE user_id = ? AND year = 2024                                │
│  ✓ WHERE user_id = ? AND year = 2024 AND month = 1                  │
│  ✓ WHERE user_id = ? AND year = 2024 AND month >= 1 AND month <= 6  │
│  ✓ WHERE user_id = ? AND year = 2024 AND month = 1 AND day = 15     │
│                                                                      │
│  ✗ Not Allowed:                                                      │
│  ✗ WHERE year = 2024                    (missing partition key)     │
│  ✗ WHERE user_id = ? AND month = 1      (skipped year)              │
│  ✗ WHERE user_id = ? AND day = 15       (skipped year, month)       │
│  ✗ WHERE user_id = ? AND year > 2023    (range not on last CK)      │
└─────────────────────────────────────────────────────────────────────┘
```

### ALLOW FILTERING

```sql
-- ALLOW FILTERING enables queries that don't follow rules
-- WARNING: Can be very expensive!

-- Without ALLOW FILTERING (error)
SELECT * FROM users WHERE email = 'john@example.com';
-- Error: Cannot execute query with ALLOW FILTERING

-- With ALLOW FILTERING (works but dangerous)
SELECT * FROM users
WHERE email = 'john@example.com'
ALLOW FILTERING;
-- Scans entire table!

-- Better solution: Create lookup table
CREATE TABLE users_by_email (
    email TEXT PRIMARY KEY,
    user_id UUID
);
-- Then query: SELECT * FROM users_by_email WHERE email = ?
```

---

## 4. Secondary Indexes

### Creating Indexes

```sql
-- Regular secondary index
CREATE INDEX ON users(email);

-- Named index
CREATE INDEX users_email_idx ON users(email);

-- Index on collection values
CREATE INDEX ON users(KEYS(settings));   -- Map keys
CREATE INDEX ON users(VALUES(tags));     -- Set/List values
CREATE INDEX ON users(ENTRIES(settings)); -- Map entries
CREATE INDEX ON users(FULL(address));    -- Frozen collection

-- SASI Index (more powerful, deprecated in favor of SAI)
CREATE CUSTOM INDEX ON users(username)
USING 'org.apache.cassandra.index.sasi.SASIIndex'
WITH OPTIONS = {
    'mode': 'CONTAINS',
    'analyzer_class': 'org.apache.cassandra.index.sasi.analyzer.StandardAnalyzer',
    'case_sensitive': 'false'
};

-- Query with SASI
SELECT * FROM users WHERE username LIKE '%john%';
```

### SAI (Storage-Attached Index) - Cassandra 5.0+

```sql
-- SAI provides better performance than legacy secondary indexes
CREATE INDEX ON users(email) USING 'sai';

-- SAI options
CREATE INDEX ON users(email) USING 'sai'
WITH OPTIONS = {'case_sensitive': 'false'};

-- Query
SELECT * FROM users WHERE email = 'john@example.com';
```

### Index Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Secondary Index Guidelines                        │
│                                                                      │
│  Good Use Cases:                                                     │
│  ✓ Low-cardinality columns (status, category)                       │
│  ✓ Combined with partition key                                      │
│  ✓ Small result sets                                                │
│                                                                      │
│  Avoid:                                                              │
│  ✗ High-cardinality columns (email, user_id)                        │
│  ✗ Frequently updated columns                                       │
│  ✗ Large result sets                                                │
│  ✗ As primary query mechanism                                       │
│                                                                      │
│  Prefer denormalized tables for:                                     │
│  • High-throughput queries                                          │
│  • Large result sets                                                │
│  • Production workloads                                             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Batch Statements

### Logged Batches

```sql
-- Logged batch (atomic, but expensive)
BEGIN BATCH
    INSERT INTO users (user_id, username, email)
    VALUES (?, 'john', 'john@example.com');

    INSERT INTO users_by_email (email, user_id)
    VALUES ('john@example.com', ?);

    INSERT INTO users_by_username (username, user_id)
    VALUES ('john', ?);
APPLY BATCH;

-- Use for maintaining multiple tables with same data
-- All statements succeed or all fail (atomic)
```

### Unlogged Batches

```sql
-- Unlogged batch (for same partition only)
BEGIN UNLOGGED BATCH
    INSERT INTO user_events (user_id, event_time, event_type)
    VALUES (?, '2024-01-15 10:00:00', 'login');

    INSERT INTO user_events (user_id, event_time, event_type)
    VALUES (?, '2024-01-15 10:05:00', 'click');

    INSERT INTO user_events (user_id, event_time, event_type)
    VALUES (?, '2024-01-15 10:10:00', 'logout');
APPLY BATCH;

-- More efficient when all rows are same partition
-- No coordination overhead
```

### Counter Batches

```sql
-- Counter batch
BEGIN COUNTER BATCH
    UPDATE page_stats SET view_count = view_count + 1 WHERE page_id = ?;
    UPDATE user_stats SET page_views = page_views + 1 WHERE user_id = ?;
APPLY BATCH;
```

### Batch Guidelines

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Batch Best Practices                              │
│                                                                      │
│  DO use batches for:                                                 │
│  ✓ Keeping denormalized tables in sync                              │
│  ✓ Multiple writes to same partition (unlogged)                     │
│  ✓ Atomic updates across related tables                             │
│                                                                      │
│  DON'T use batches for:                                              │
│  ✗ "Bulk loading" (use async writes instead)                        │
│  ✗ Performance optimization                                         │
│  ✗ Batching unrelated writes                                        │
│  ✗ Large batches (> 50 statements or > 50KB)                        │
│                                                                      │
│  Performance:                                                        │
│  • Logged batches: Extra coordination, higher latency               │
│  • Unlogged batches: Same partition = efficient                     │
│  • Large batches: Timeouts, memory pressure                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Lightweight Transactions (LWT)

### Compare and Set

```sql
-- Insert if not exists
INSERT INTO users (user_id, username, email)
VALUES (uuid(), 'unique_user', 'user@example.com')
IF NOT EXISTS;

-- Returns: [applied]
-- true if inserted, false if existed

-- Conditional update
UPDATE users
SET email = 'new@example.com'
WHERE user_id = ?
IF email = 'old@example.com';

-- Multiple conditions
UPDATE users
SET email = 'new@example.com', status = 'verified'
WHERE user_id = ?
IF email = 'old@example.com' AND status = 'pending';
```

### LWT Patterns

```sql
-- Distributed lock (simplified)
INSERT INTO locks (resource_id, owner, acquired_at)
VALUES ('my-resource', 'worker-1', toTimestamp(now()))
IF NOT EXISTS
USING TTL 60;

-- Release lock
DELETE FROM locks
WHERE resource_id = 'my-resource'
IF owner = 'worker-1';

-- Optimistic locking with version
UPDATE documents
SET content = 'new content', version = 6
WHERE doc_id = ?
IF version = 5;
```

### LWT Considerations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LWT Trade-offs                                    │
│                                                                      │
│  Benefits:                                                           │
│  ✓ Linearizable consistency (Paxos)                                 │
│  ✓ Compare-and-swap semantics                                       │
│  ✓ Prevents race conditions                                         │
│                                                                      │
│  Costs:                                                              │
│  ✗ 4x latency vs regular writes                                     │
│  ✗ Requires contacting multiple replicas                            │
│  ✗ Can cause contention on hot rows                                 │
│  ✗ Not designed for high throughput                                 │
│                                                                      │
│  Best Practices:                                                     │
│  • Use sparingly (not for every write)                              │
│  • Avoid on hot partitions                                          │
│  • Consider application-level solutions                             │
│  • Batch LWTs on same partition when possible                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Prepared Statements

### Using Prepared Statements

```python
from cassandra.cluster import Cluster

cluster = Cluster(['localhost'])
session = cluster.connect('my_app')

# Prepare statement (do once)
insert_user = session.prepare("""
    INSERT INTO users (user_id, username, email, created_at)
    VALUES (?, ?, ?, ?)
""")

# Execute many times (efficient)
session.execute(insert_user, [user_id, 'john', 'john@example.com', datetime.now()])
session.execute(insert_user, [user_id2, 'jane', 'jane@example.com', datetime.now()])

# Prepared with named parameters
select_user = session.prepare("""
    SELECT * FROM users WHERE user_id = :user_id
""")
session.execute(select_user, {'user_id': user_id})
```

### Benefits

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Prepared Statement Benefits                       │
│                                                                      │
│  Performance:                                                        │
│  • Query parsed once, executed many times                           │
│  • Reduced network traffic (send ID, not query)                     │
│  • Automatic token awareness (route to correct node)                │
│                                                                      │
│  Security:                                                           │
│  • Prevents CQL injection                                           │
│  • Type validation on parameters                                    │
│                                                                      │
│  Usage Pattern:                                                      │
│  prepare() → returns PreparedStatement (cache this!)                │
│  bind() → returns BoundStatement with values                        │
│  execute() → runs the statement                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Aggregations and Functions

### Built-in Functions

```sql
-- UUID/Time functions
SELECT uuid(), now(), toTimestamp(now()), toDate(now());
SELECT unixTimestampOf(now());
SELECT dateOf(now());  -- Deprecated, use toDate

-- Type conversions
SELECT blobAsText(textAsBlob('hello'));
SELECT typeAsBlob(my_column);

-- Token function
SELECT token(user_id) FROM users WHERE user_id = ?;

-- TTL and Writetime
SELECT TTL(email), WRITETIME(email) FROM users WHERE user_id = ?;
```

### Aggregate Functions

```sql
-- Count
SELECT COUNT(*) FROM users WHERE user_id = ?;

-- Sum, Avg, Min, Max
SELECT SUM(amount), AVG(amount), MIN(amount), MAX(amount)
FROM orders WHERE customer_id = ?;

-- Note: Aggregations work within partition
-- Cross-partition aggregations require ALLOW FILTERING (expensive)
```

### User-Defined Functions (UDF)

```sql
-- Enable in cassandra.yaml: enable_user_defined_functions: true

-- Create UDF
CREATE FUNCTION calculate_discount(price DOUBLE, discount_percent INT)
CALLED ON NULL INPUT
RETURNS DOUBLE
LANGUAGE java
AS 'return price * (1 - discount_percent / 100.0);';

-- Use in query
SELECT product_id, calculate_discount(price, 10) AS discounted_price
FROM products WHERE category = ?;

-- User-Defined Aggregate
CREATE AGGREGATE avg_price(DOUBLE)
SFUNC avgState
STYPE tuple<DOUBLE, INT>
FINALFUNC avgFinal
INITCOND (0.0, 0);
```

---

## Summary

| Operation | Key Points |
|-----------|------------|
| SELECT | Must include partition key, follow clustering order |
| INSERT | Use TTL for temporary data, IF NOT EXISTS for safety |
| UPDATE | Collection operations, conditional with IF |
| DELETE | Can delete columns, ranges, conditionally |
| BATCH | Use for related writes, not bulk loading |
| LWT | Expensive but provides linearizability |

---

## Best Practices

```
Queries:
✓ Always use prepared statements
✓ Include partition key in WHERE
✓ Follow clustering key order
✓ Use LIMIT for large partitions
✗ Avoid ALLOW FILTERING in production

Writes:
✓ Use async writes for throughput
✓ Consider TTL for temporary data
✓ Use unlogged batches for same partition
✗ Avoid large batches (> 50 statements)

LWT:
✓ Use for unique constraints
✓ Use for optimistic locking
✗ Avoid on hot partitions
✗ Don't use for every write
```
