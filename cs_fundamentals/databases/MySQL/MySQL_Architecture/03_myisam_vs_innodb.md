# MyISAM vs InnoDB

## Learning Objectives
- Understand the key differences between MyISAM and InnoDB
- Learn when to use each storage engine
- Master migration strategies from MyISAM to InnoDB
- Understand the architectural differences

---

## 1. Feature Comparison Overview

| Feature | InnoDB | MyISAM |
|---------|--------|--------|
| **ACID Compliance** | Yes | No |
| **Transactions** | Yes | No |
| **Row-level Locking** | Yes | No (table-level only) |
| **Foreign Keys** | Yes | No |
| **Crash Recovery** | Automatic | Manual repair |
| **MVCC** | Yes | No |
| **Full-text Search** | Yes (5.6+) | Yes |
| **Spatial Indexes** | Yes (5.7+) | Yes |
| **Data Caching** | Buffer pool | OS file cache |
| **Index Caching** | Buffer pool | Key buffer |
| **Compression** | Yes | Yes (different) |
| **Encryption** | Yes | No |

---

## 2. Architecture Differences

### InnoDB Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        InnoDB Table                              │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Clustered Index                         │  │
│  │  (Primary Key = Data Organization)                         │  │
│  │                                                            │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  PK=1 → [id=1, name='Alice', email='a@test.com']    │  │  │
│  │  │  PK=2 → [id=2, name='Bob', email='b@test.com']      │  │  │
│  │  │  PK=3 → [id=3, name='Carol', email='c@test.com']    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Secondary Index (idx_name)                    │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  'Alice' → PK=1 (pointer to clustered index)        │  │  │
│  │  │  'Bob' → PK=2                                       │  │  │
│  │  │  'Carol' → PK=3                                     │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  File: tablename.ibd (data + indexes together)                   │
└─────────────────────────────────────────────────────────────────┘
```

### MyISAM Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MyISAM Table                              │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Data File (tablename.MYD)                     │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  Row 1: [id=1, name='Alice', email='a@test.com']    │  │  │
│  │  │  Row 2: [id=2, name='Bob', email='b@test.com']      │  │  │
│  │  │  Row 3: [id=3, name='Carol', email='c@test.com']    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Index File (tablename.MYI)                    │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  Primary Index: id → row pointer (file offset)      │  │  │
│  │  │  Secondary: name → row pointer (file offset)        │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  File: tablename.frm (table definition, pre-8.0)                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Locking Mechanisms

### InnoDB Row-Level Locking

```sql
-- InnoDB locks only affected rows
-- Session 1:
BEGIN;
UPDATE users SET status = 'active' WHERE id = 1;
-- Only row id=1 is locked

-- Session 2 (concurrent):
UPDATE users SET status = 'inactive' WHERE id = 2;
-- Succeeds immediately! Different row.

-- Session 1:
COMMIT;
```

### MyISAM Table-Level Locking

```sql
-- MyISAM locks entire table
-- Session 1:
UPDATE users SET status = 'active' WHERE id = 1;
-- ENTIRE table is locked during update

-- Session 2 (concurrent):
UPDATE users SET status = 'inactive' WHERE id = 2;
-- BLOCKED! Waits for table lock even though different row.

-- Session 2 waits until Session 1 completes
```

### Lock Escalation Impact

```
┌─────────────────────────────────────────────────────────────────┐
│                   Concurrent Write Performance                   │
│                                                                  │
│  InnoDB (row locks):                                             │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Thread 1: ████████                                       │   │
│  │  Thread 2:   ████████                                     │   │
│  │  Thread 3:     ████████                                   │   │
│  │  Thread 4:       ████████                                 │   │
│  │  (Parallel execution on different rows)                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  MyISAM (table locks):                                           │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Thread 1: ████████                                       │   │
│  │  Thread 2:         ████████                               │   │
│  │  Thread 3:                 ████████                       │   │
│  │  Thread 4:                         ████████               │   │
│  │  (Serial execution - table lock contention)               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Transaction Support

### InnoDB Transactions

```sql
-- InnoDB supports full ACID transactions
BEGIN;

INSERT INTO accounts (user_id, balance) VALUES (1, 1000);
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
INSERT INTO transactions (user_id, amount, type) VALUES (1, -100, 'debit');

-- If any statement fails, rollback all changes
ROLLBACK;

-- Or commit all changes atomically
COMMIT;

-- Savepoints
BEGIN;
UPDATE accounts SET balance = balance - 50 WHERE user_id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance - 50 WHERE user_id = 2;
-- Oops, wrong account
ROLLBACK TO SAVEPOINT sp1;
-- First update preserved
COMMIT;
```

### MyISAM - No Transactions

```sql
-- MyISAM has no transaction support
INSERT INTO accounts (user_id, balance) VALUES (1, 1000);  -- Committed immediately
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;  -- Committed immediately

-- No way to rollback!
-- If UPDATE fails midway on multi-row update, partial changes are kept

-- Workaround: LOCK TABLES (not true transaction)
LOCK TABLES accounts WRITE, transactions WRITE;
INSERT INTO accounts (user_id, balance) VALUES (1, 1000);
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
UNLOCK TABLES;
```

---

## 5. Crash Recovery

### InnoDB Automatic Recovery

```
┌─────────────────────────────────────────────────────────────────┐
│                   InnoDB Crash Recovery                          │
│                                                                  │
│  1. Server crashes during transaction                            │
│                                                                  │
│  2. On restart:                                                  │
│     ┌────────────────────────────────────────────────────────┐  │
│     │  a) Read redo logs                                      │  │
│     │  b) Redo committed transactions (REDO phase)           │  │
│     │  c) Undo uncommitted transactions (UNDO phase)         │  │
│     └────────────────────────────────────────────────────────┘  │
│                                                                  │
│  3. Database returns to consistent state automatically           │
│                                                                  │
│  No data loss for committed transactions!                        │
└─────────────────────────────────────────────────────────────────┘
```

```sql
-- InnoDB recovery is automatic
-- Just restart MySQL and recovery happens

-- Check recovery status in error log
-- [InnoDB] Starting crash recovery
-- [InnoDB] Redo log recovery complete
```

### MyISAM Manual Recovery

```bash
# MyISAM requires manual repair after crash
myisamchk --recover /var/lib/mysql/mydb/mytable.MYI

# Or within MySQL
REPAIR TABLE mytable;

# Check for corruption
CHECK TABLE mytable;
myisamchk --check /var/lib/mysql/mydb/mytable.MYI

# Extended repair for severe corruption
myisamchk --safe-recover --extend-check mytable.MYI
```

---

## 6. Performance Characteristics

### Read Performance

```sql
-- MyISAM advantages for reads:
-- 1. Smaller index size (no transaction overhead)
-- 2. No MVCC overhead
-- 3. Simple table scans can be faster

-- InnoDB advantages for reads:
-- 1. Buffer pool caches data AND indexes
-- 2. Consistent reads via MVCC
-- 3. Better concurrent read performance
```

### Write Performance

```sql
-- InnoDB write advantages:
-- 1. Row-level locking allows concurrent writes
-- 2. Group commit for better write throughput
-- 3. Change buffer for secondary index updates

-- MyISAM write limitations:
-- 1. Table lock on any write
-- 2. No buffering of writes
-- 3. Concurrent writes serialize

-- Benchmark: 1000 concurrent INSERT operations
-- InnoDB: ~3 seconds (parallel)
-- MyISAM: ~15 seconds (serialized by table locks)
```

### Count Performance

```sql
-- MyISAM stores row count
SELECT COUNT(*) FROM myisam_table;  -- Instant (reads stored count)

-- InnoDB must scan (for accurate count with MVCC)
SELECT COUNT(*) FROM innodb_table;  -- Requires index scan

-- Workaround for InnoDB
-- Use covering index
SELECT COUNT(id) FROM innodb_table;  -- Uses primary key index

-- Or maintain counter table
CREATE TABLE row_counts (
    table_name VARCHAR(64) PRIMARY KEY,
    row_count BIGINT
);
```

---

## 7. Memory Usage

### InnoDB Buffer Pool

```sql
-- InnoDB caches everything in buffer pool
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- Recommended: 70-80% of available RAM
SET GLOBAL innodb_buffer_pool_size = 12G;

-- Buffer pool caches:
-- - Data pages
-- - Index pages
-- - Undo logs
-- - Change buffer
-- - Adaptive hash index
```

### MyISAM Key Buffer

```sql
-- MyISAM only caches indexes, not data
SHOW VARIABLES LIKE 'key_buffer_size';

-- Default: 8MB (usually too small)
SET GLOBAL key_buffer_size = 512M;

-- Data reads go through OS file cache
-- Less predictable performance
```

---

## 8. When to Use Each Engine

### Use InnoDB When:

1. **Transactional integrity required**
   - Financial applications
   - E-commerce
   - Any multi-table updates

2. **Concurrent access**
   - Web applications with many simultaneous users
   - Mixed read/write workloads

3. **Data durability critical**
   - Production databases
   - Mission-critical applications

4. **Foreign key relationships**
   - Relational data models
   - Referential integrity needs

```sql
-- InnoDB is default since MySQL 5.5
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
) ENGINE=InnoDB;
```

### Use MyISAM When (Legacy/Specific Cases):

1. **Read-heavy with infrequent writes**
   - Archive tables
   - Logging (append-only)

2. **Full-text search (pre-5.6)**
   - Legacy search implementations

3. **Geographic data (pre-5.7)**
   - Legacy spatial applications

4. **COUNT(*) performance critical**
   - Reporting on table sizes

```sql
-- MyISAM for specific use cases
CREATE TABLE access_log (
    id INT PRIMARY KEY AUTO_INCREMENT,
    timestamp DATETIME,
    url VARCHAR(255),
    INDEX (timestamp)
) ENGINE=MyISAM;
```

---

## 9. Migration from MyISAM to InnoDB

### Single Table Migration

```sql
-- Convert table engine
ALTER TABLE mytable ENGINE = InnoDB;

-- This operation:
-- 1. Creates new InnoDB table
-- 2. Copies all data
-- 3. Drops old MyISAM table
-- 4. Renames new table

-- Check progress (MySQL 8.0)
SELECT * FROM performance_schema.events_stages_current;
```

### Batch Migration Script

```sql
-- Find all MyISAM tables
SELECT
    TABLE_SCHEMA,
    TABLE_NAME,
    ENGINE,
    TABLE_ROWS,
    ROUND(DATA_LENGTH/1024/1024, 2) AS data_mb
FROM information_schema.TABLES
WHERE ENGINE = 'MyISAM'
AND TABLE_SCHEMA NOT IN ('mysql', 'information_schema', 'performance_schema');

-- Generate ALTER statements
SELECT CONCAT('ALTER TABLE `', TABLE_SCHEMA, '`.`', TABLE_NAME, '` ENGINE=InnoDB;')
FROM information_schema.TABLES
WHERE ENGINE = 'MyISAM'
AND TABLE_SCHEMA = 'mydb';
```

### pt-online-schema-change (Large Tables)

```bash
# Percona Toolkit for online migration
pt-online-schema-change \
    --alter "ENGINE=InnoDB" \
    --execute \
    --no-drop-old-table \
    --chunk-size=1000 \
    D=mydb,t=large_table

# Benefits:
# - Minimal locking
# - Progress tracking
# - Pause/resume capability
```

### Migration Considerations

```sql
-- Before migration:
-- 1. Check for implicit behaviors
--    MyISAM: Inserts to end of table (no gaps)
--    InnoDB: Uses AUTO_INCREMENT gaps

-- 2. Check COUNT(*) usage
--    May need to add covering indexes or counter tables

-- 3. Review FULLTEXT indexes (pre-5.6)
--    May need recreation

-- 4. Check table locks usage
--    LOCK TABLES works differently with InnoDB

-- 5. Disk space
--    InnoDB tables may be larger (undo info, etc.)
SELECT
    ENGINE,
    SUM(DATA_LENGTH)/1024/1024 AS data_mb,
    SUM(INDEX_LENGTH)/1024/1024 AS index_mb
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb'
GROUP BY ENGINE;
```

---

## 10. Configuration Comparison

### InnoDB Settings

```ini
[mysqld]
# Storage
innodb_buffer_pool_size = 12G
innodb_log_file_size = 1G
innodb_file_per_table = ON

# Durability
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1

# Concurrency
innodb_thread_concurrency = 0
innodb_read_io_threads = 4
innodb_write_io_threads = 4
```

### MyISAM Settings

```ini
[mysqld]
# Index cache
key_buffer_size = 512M

# Bulk insert optimization
bulk_insert_buffer_size = 64M

# Repair settings
myisam_sort_buffer_size = 64M
myisam_max_sort_file_size = 10G

# Concurrent inserts
concurrent_insert = 2
```

---

## Summary

| Aspect | Choose InnoDB | Choose MyISAM |
|--------|---------------|---------------|
| Transactions | Required | Not needed |
| Concurrent writes | High | Low |
| Data integrity | Critical | Acceptable loss |
| Foreign keys | Required | Not needed |
| Crash recovery | Automatic needed | Manual OK |
| COUNT(*) speed | Not critical | Critical |
| Full-text (pre-5.6) | Not needed | Required |

**Recommendation**: Use InnoDB for all new projects. MyISAM is deprecated and removed from MySQL 8.0's system tables.

---

## Further Reading

- MySQL Storage Engine comparison documentation
- Percona Blog - MyISAM to InnoDB migration
- "High Performance MySQL" - Storage engine selection
