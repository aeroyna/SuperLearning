# Hash Indexes in MySQL

## Learning Objectives
- Understand hash index structure and operations
- Learn when hash indexes are appropriate
- Compare hash vs B-Tree indexes
- Understand adaptive hash index in InnoDB

---

## 1. Hash Index Fundamentals

### How Hash Indexes Work

```
┌─────────────────────────────────────────────────────────────────┐
│                      Hash Index Structure                        │
│                                                                  │
│  Key: 'alice@test.com'                                           │
│       ↓                                                          │
│  Hash Function: hash('alice@test.com') = 4827                    │
│       ↓                                                          │
│  Hash Table:                                                     │
│  ┌─────────┬────────────────────────────────────────────┐       │
│  │ Bucket  │ Entries                                     │       │
│  ├─────────┼────────────────────────────────────────────┤       │
│  │    0    │ → NULL                                      │       │
│  │    1    │ → [hash=1, 'bob@test.com', ptr→row5]       │       │
│  │   ...   │ → ...                                       │       │
│  │  4827   │ → [hash=4827, 'alice@test.com', ptr→row12] │ ← HIT│
│  │   ...   │ → ...                                       │       │
│  │  9999   │ → [hash=9999, 'dave@test.com', ptr→row3]   │       │
│  └─────────┴────────────────────────────────────────────┘       │
│                                                                  │
│  Lookup: O(1) average case                                       │
└─────────────────────────────────────────────────────────────────┘
```

### Hash Index Properties

| Property | Description |
|----------|-------------|
| **Time Complexity** | O(1) average for exact match |
| **Range Queries** | Not supported |
| **Ordering** | Not preserved |
| **Prefix Matching** | Not supported |
| **Collision Handling** | Chaining or open addressing |

---

## 2. Hash Indexes in MySQL

### MEMORY/HEAP Engine

```sql
-- Create table with MEMORY engine (supports hash indexes)
CREATE TABLE session_cache (
    session_id CHAR(36) PRIMARY KEY,
    user_id INT,
    data TEXT,
    expires_at DATETIME,
    INDEX USING HASH (user_id)  -- Explicit hash index
) ENGINE = MEMORY;

-- MEMORY engine defaults to hash indexes
-- B-Tree available with USING BTREE

-- Comparison
CREATE TABLE cache_btree (
    id INT PRIMARY KEY,
    key_name VARCHAR(100),
    INDEX USING BTREE (key_name)  -- B-Tree on MEMORY table
) ENGINE = MEMORY;

CREATE TABLE cache_hash (
    id INT PRIMARY KEY,
    key_name VARCHAR(100),
    INDEX USING HASH (key_name)   -- Hash index
) ENGINE = MEMORY;
```

### MEMORY Engine Limitations

```sql
-- MEMORY engine constraints:
-- 1. Data lost on server restart
-- 2. Table-level locking only
-- 3. No BLOB/TEXT columns
-- 4. Fixed row format (wastes space for VARCHAR)
-- 5. Limited by max_heap_table_size

SHOW VARIABLES LIKE 'max_heap_table_size';
SET max_heap_table_size = 256 * 1024 * 1024;  -- 256MB
```

---

## 3. Hash vs B-Tree Comparison

### When to Use Hash Index

```sql
-- Hash indexes excel at exact match lookups:
SELECT * FROM cache WHERE key = 'user_123_profile';  -- O(1)

-- Perfect for:
-- - Session lookups by ID
-- - Cache key lookups
-- - Exact match on high-cardinality columns
```

### When NOT to Use Hash Index

```sql
-- Hash indexes CANNOT do:

-- 1. Range queries
SELECT * FROM orders WHERE created_at > '2024-01-01';  -- ✗

-- 2. Ordering
SELECT * FROM users ORDER BY email;  -- ✗

-- 3. Prefix matching
SELECT * FROM users WHERE email LIKE 'john%';  -- ✗

-- 4. Partial key matching (composite index)
INDEX (a, b, c) → Cannot query just 'a'  -- ✗
```

### Performance Comparison

```
┌────────────────────────────────────────────────────────────────┐
│              Hash vs B-Tree Performance                         │
│                                                                 │
│  Operation              Hash Index    B-Tree Index              │
│  ─────────────────────────────────────────────────────────────  │
│  Exact match (=)        O(1)          O(log n)                  │
│  Range (<, >, BETWEEN)  Not possible  O(log n + k)              │
│  ORDER BY               Not possible  O(log n) start            │
│  LIKE 'prefix%'         Not possible  O(log n + k)              │
│  MIN/MAX                Not possible  O(log n)                  │
│                                                                 │
│  Best Use Case:                                                 │
│  Hash:   Pure key-value lookups, caching                        │
│  B-Tree: General purpose, range queries, ordering               │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. InnoDB Adaptive Hash Index

InnoDB automatically builds hash indexes for frequently accessed B-Tree index pages.

### How Adaptive Hash Index Works

```
┌─────────────────────────────────────────────────────────────────┐
│                 Adaptive Hash Index (AHI)                        │
│                                                                  │
│  Normal B-Tree Lookup:                                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Query: WHERE id = 42                                         ││
│  │                                                              ││
│  │ Root → Internal → Internal → Leaf → Row                      ││
│  │ (3-4 page accesses)                                          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  After AHI builds hash:                                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Query: WHERE id = 42                                         ││
│  │                                                              ││
│  │ Adaptive Hash Index → Direct to Leaf → Row                   ││
│  │ (1 hash lookup + 1 page access)                              ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  AHI is automatic and transparent to user                        │
└─────────────────────────────────────────────────────────────────┘
```

### AHI Configuration

```sql
-- Check if AHI is enabled
SHOW VARIABLES LIKE 'innodb_adaptive_hash_index';

-- Enable/disable (requires restart or dynamic in 5.7+)
SET GLOBAL innodb_adaptive_hash_index = ON;
SET GLOBAL innodb_adaptive_hash_index = OFF;

-- AHI partitions (reduce contention)
SHOW VARIABLES LIKE 'innodb_adaptive_hash_index_parts';
-- Default: 8 partitions
```

### Monitoring AHI

```sql
-- AHI statistics in SHOW ENGINE INNODB STATUS
SHOW ENGINE INNODB STATUS\G

/*
Look for:
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Hash table size 34679, node heap has 1 buffer(s)
Hash table size 34679, node heap has 0 buffer(s)
...
0.00 hash searches/s, 0.00 non-hash searches/s

High ratio of hash/non-hash searches = AHI is effective
*/

-- Check AHI hit rate
SELECT
    NAME,
    COUNT,
    STATUS
FROM information_schema.INNODB_METRICS
WHERE NAME LIKE '%adaptive_hash%';
```

### When to Disable AHI

```sql
-- Consider disabling AHI when:
-- 1. High contention on AHI latch (visible in semaphore waits)
-- 2. Random access patterns (AHI cache thrashing)
-- 3. Very large tables with uniform access

-- Check for AHI contention
SHOW ENGINE INNODB STATUS\G
/*
Look for:
--Thread XXX has waited at btr0sea.cc line XXX for XX seconds
*/

-- Benchmark with and without AHI for your workload
```

---

## 5. NDB Cluster Hash Indexes

MySQL NDB Cluster uses hash indexes for primary key lookups.

```sql
-- NDB Cluster creates hash index on PRIMARY KEY by default
CREATE TABLE ndb_table (
    id INT PRIMARY KEY,       -- Automatic hash index
    name VARCHAR(100),
    email VARCHAR(100),
    INDEX idx_email (email)   -- Ordered index (T-tree)
) ENGINE = NDBCLUSTER;

-- NDB uses hash for data distribution across nodes
-- Primary key determines which node stores the row
```

---

## 6. Simulating Hash Indexes in InnoDB

When you need hash-like performance in InnoDB:

### Using Generated Hash Column

```sql
-- Add a hash column for fast lookups
CREATE TABLE urls (
    id INT PRIMARY KEY AUTO_INCREMENT,
    url VARCHAR(2048),
    url_hash BIGINT UNSIGNED AS (CRC32(url)) STORED,
    content TEXT,
    INDEX idx_hash (url_hash)
);

-- Fast lookup using hash
SELECT * FROM urls
WHERE url_hash = CRC32('https://example.com/long/path/to/page')
  AND url = 'https://example.com/long/path/to/page';

-- The hash narrows down candidates quickly
-- The url comparison ensures exact match (handles collisions)
```

### Using MD5/SHA for Uniqueness

```sql
-- For longer strings where collisions must be rare
CREATE TABLE sessions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    session_token VARCHAR(64),
    token_hash BINARY(16) AS (UNHEX(MD5(session_token))) STORED,
    user_id INT,
    created_at DATETIME,
    UNIQUE INDEX idx_token_hash (token_hash)
);

-- Lookup by token
SELECT * FROM sessions
WHERE token_hash = UNHEX(MD5('abc123token'));
```

---

## 7. Hash Collisions

### Understanding Collisions

```
┌─────────────────────────────────────────────────────────────────┐
│                     Hash Collision                               │
│                                                                  │
│  hash('alice@test.com') = 4827                                   │
│  hash('bob@example.org') = 4827  ← Same hash!                    │
│                                                                  │
│  Bucket 4827:                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ → [alice@test.com, ptr→row12]                               │ │
│  │   → [bob@example.org, ptr→row45]                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Lookup 'alice@test.com':                                        │
│  1. Hash to bucket 4827                                          │
│  2. Search chain, compare actual values                          │
│  3. Return row12                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Handling Collisions in Application

```sql
-- Always verify actual value after hash lookup
SELECT * FROM urls
WHERE url_hash = CRC32('https://example.com')
  AND url = 'https://example.com';  -- Verify actual URL

-- Without verification, you might get wrong row on collision!
```

---

## 8. Practical Examples

### Session Store

```sql
-- MEMORY table for session cache
CREATE TABLE session_store (
    session_id CHAR(36) NOT NULL,
    user_id INT NOT NULL,
    data JSON,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (session_id) USING HASH,  -- Hash for O(1) lookup
    INDEX idx_user (user_id) USING HASH
) ENGINE = MEMORY;

-- Fast session lookup
SELECT data FROM session_store WHERE session_id = 'uuid-string';

-- Cleanup: Copy to InnoDB periodically for persistence
CREATE TABLE session_archive LIKE session_store ENGINE = InnoDB;
INSERT INTO session_archive SELECT * FROM session_store;
```

### Rate Limiting Cache

```sql
-- Track API rate limits
CREATE TABLE rate_limits (
    client_key VARCHAR(64) NOT NULL,
    request_count INT DEFAULT 1,
    window_start DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (client_key) USING HASH
) ENGINE = MEMORY;

-- Increment or insert
INSERT INTO rate_limits (client_key)
VALUES ('api_key_123')
ON DUPLICATE KEY UPDATE
    request_count = IF(
        window_start < NOW() - INTERVAL 1 MINUTE,
        1,  -- Reset counter
        request_count + 1
    ),
    window_start = IF(
        window_start < NOW() - INTERVAL 1 MINUTE,
        NOW(),
        window_start
    );
```

---

## Summary

| Aspect | Hash Index | B-Tree Index |
|--------|------------|--------------|
| Exact match | O(1) | O(log n) |
| Range queries | ✗ | ✓ |
| Sorting | ✗ | ✓ |
| Prefix search | ✗ | ✓ |
| Storage Engine | MEMORY, NDB | All |
| InnoDB automatic | Adaptive Hash Index | Primary |

### Key Takeaways

1. **Hash indexes** are limited to MEMORY engine in MySQL
2. **InnoDB Adaptive Hash Index** provides hash-like speed automatically
3. **Simulate hash indexes** in InnoDB with generated columns
4. **B-Tree is more versatile** and usually the right choice
5. **Hash excels** at pure key-value lookups

---

## Further Reading

- MySQL MEMORY Storage Engine documentation
- InnoDB Adaptive Hash Index internals
- Hash table data structure theory
