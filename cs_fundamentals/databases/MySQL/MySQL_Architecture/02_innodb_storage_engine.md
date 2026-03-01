# InnoDB Storage Engine

## Learning Objectives
- Understand InnoDB architecture and components
- Master buffer pool management
- Learn InnoDB transaction and locking mechanisms
- Understand redo/undo logging and crash recovery

---

## 1. InnoDB Overview

InnoDB is MySQL's default storage engine since version 5.5, providing:

- Full ACID compliance
- Row-level locking
- Multi-Version Concurrency Control (MVCC)
- Foreign key constraints
- Automatic crash recovery
- Clustered indexes

---

## 2. InnoDB Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         InnoDB Architecture                              │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                        IN-MEMORY STRUCTURES                         │ │
│  │  ┌────────────────────────────────────────────────────────────┐    │ │
│  │  │                      Buffer Pool                            │    │ │
│  │  │  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌────────────┐ │    │ │
│  │  │  │ Data     │  │ Index    │  │ Undo      │  │ Adaptive   │ │    │ │
│  │  │  │ Pages    │  │ Pages    │  │ Pages     │  │ Hash Index │ │    │ │
│  │  │  └──────────┘  └──────────┘  └───────────┘  └────────────┘ │    │ │
│  │  └────────────────────────────────────────────────────────────┘    │ │
│  │                                                                     │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────────┐    │ │
│  │  │ Change Buffer  │  │ Log Buffer     │  │ Adaptive Hash      │    │ │
│  │  │ (Insert Buffer)│  │ (Redo Log Buf) │  │ Index              │    │ │
│  │  └────────────────┘  └────────────────┘  └────────────────────┘    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                    │                                     │
│                                    ▼                                     │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                        ON-DISK STRUCTURES                           │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────────┐    │ │
│  │  │ System         │  │ Redo Log       │  │ Undo              │    │ │
│  │  │ Tablespace     │  │ Files          │  │ Tablespaces       │    │ │
│  │  │ (ibdata1)      │  │ (ib_logfile*)  │  │ (undo_*)          │    │ │
│  │  └────────────────┘  └────────────────┘  └────────────────────┘    │ │
│  │                                                                     │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌────────────────────┐    │ │
│  │  │ Table          │  │ Temporary      │  │ Doublewrite        │    │ │
│  │  │ Tablespaces    │  │ Tablespace     │  │ Buffer             │    │ │
│  │  │ (*.ibd)        │  │ (ibtmp1)       │  │ (dblwr files)      │    │ │
│  │  └────────────────┘  └────────────────┘  └────────────────────┘    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Buffer Pool

The buffer pool is InnoDB's main memory area for caching data and indexes.

### Buffer Pool Structure

```
┌────────────────────────────────────────────────────────────────┐
│                        Buffer Pool                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Page Hash Table                       │   │
│  │   (space_id, page_no) → buffer page pointer              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      LRU List                            │   │
│  │  ┌────────────────────────────────────────────────────┐ │   │
│  │  │ Young Sublist (Hot)              │ Old Sublist     │ │   │
│  │  │ [Page1][Page2][Page3]...[PageN]  │ [Old1][Old2]... │ │   │
│  │  │         (5/8 of pool)            │   (3/8)         │ │   │
│  │  └────────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌───────────────────┐  ┌───────────────────────────────────┐  │
│  │    Free List      │  │        Flush List                 │  │
│  │ [Free][Free]...   │  │ [Dirty1][Dirty2][Dirty3]...       │  │
│  └───────────────────┘  └───────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

### Buffer Pool Configuration

```sql
-- View buffer pool settings
SHOW VARIABLES LIKE 'innodb_buffer_pool%';

-- Key settings
innodb_buffer_pool_size = 12G           -- 70-80% of available RAM
innodb_buffer_pool_instances = 8        -- Parallelism (1 per GB)
innodb_buffer_pool_chunk_size = 128M    -- Resize granularity

-- Buffer pool sizing formula
-- buffer_pool_size = n * buffer_pool_chunk_size * buffer_pool_instances
```

### Buffer Pool Monitoring

```sql
-- Buffer pool status
SHOW ENGINE INNODB STATUS\G

-- Buffer pool statistics
SELECT
    POOL_ID,
    POOL_SIZE,
    FREE_BUFFERS,
    DATABASE_PAGES,
    MODIFIED_DB_PAGES AS dirty_pages,
    HIT_RATE
FROM information_schema.INNODB_BUFFER_POOL_STATS;

-- Buffer pool hit ratio calculation
SELECT
    (1 - (Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests)) * 100
    AS buffer_pool_hit_ratio
FROM (
    SELECT
        VARIABLE_VALUE AS Innodb_buffer_pool_reads
    FROM performance_schema.global_status
    WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads'
) reads,
(
    SELECT
        VARIABLE_VALUE AS Innodb_buffer_pool_read_requests
    FROM performance_schema.global_status
    WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests'
) requests;

-- Target: > 99% hit ratio
```

---

## 4. Tablespaces

### System Tablespace (ibdata1)

Contains:
- Data dictionary (pre-8.0)
- Doublewrite buffer
- Change buffer
- Undo logs (configurable)

```sql
-- System tablespace settings
SHOW VARIABLES LIKE 'innodb_data_file_path';
-- Default: ibdata1:12M:autoextend

-- Configure system tablespace
[mysqld]
innodb_data_file_path = ibdata1:1G;ibdata2:1G:autoextend
innodb_autoextend_increment = 64  -- MB
```

### File-Per-Table Tablespace

Each table has its own .ibd file:

```sql
-- Enable file-per-table (default since 5.6)
SET GLOBAL innodb_file_per_table = ON;

-- View tablespace information
SELECT
    NAME,
    SPACE,
    FILE_SIZE/1024/1024 AS size_mb,
    ALLOCATED_SIZE/1024/1024 AS allocated_mb
FROM information_schema.INNODB_TABLESPACES
WHERE NAME LIKE 'mydb/%';
```

### General Tablespaces

Shared tablespace for multiple tables:

```sql
-- Create general tablespace
CREATE TABLESPACE ts_data
    ADD DATAFILE 'ts_data.ibd'
    ENGINE = InnoDB;

-- Create table in specific tablespace
CREATE TABLE orders (
    id INT PRIMARY KEY,
    total DECIMAL(10,2)
) TABLESPACE ts_data;

-- Move table to tablespace
ALTER TABLE customers TABLESPACE ts_data;
```

---

## 5. Redo Log (Write-Ahead Log)

### Redo Log Purpose

- Crash recovery
- Durability guarantee
- Performance optimization (sequential writes)

```
┌─────────────────────────────────────────────────────────────────┐
│                      Redo Log Architecture                       │
│                                                                  │
│   Log Buffer (in memory)                                         │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ [LSN:1000][LSN:1001][LSN:1002][LSN:1003]...              │  │
│   └──────────────────────────────────────────────────────────┘  │
│                           │                                      │
│                           │ Flush (commit or interval)           │
│                           ▼                                      │
│   Redo Log Files (on disk)                                       │
│   ┌───────────────────┐  ┌───────────────────┐                  │
│   │   ib_logfile0     │──│   ib_logfile1     │ ←─┐              │
│   │   [blocks...]     │  │   [blocks...]     │   │ circular     │
│   └───────────────────┘  └───────────────────┘ ──┘              │
└─────────────────────────────────────────────────────────────────┘
```

### Redo Log Configuration

```sql
-- View redo log settings
SHOW VARIABLES LIKE 'innodb_log%';

-- Configuration (my.cnf)
[mysqld]
innodb_log_file_size = 1G        -- Size per file (1-2x buffer pool size)
innodb_log_files_in_group = 2    -- Number of files (deprecated in 8.0.30+)
innodb_log_buffer_size = 64M     -- In-memory buffer
innodb_flush_log_at_trx_commit = 1  -- Durability setting

-- MySQL 8.0.30+ uses innodb_redo_log_capacity
innodb_redo_log_capacity = 2G    -- Total redo log size
```

### Flush Settings

```sql
-- innodb_flush_log_at_trx_commit options:
-- 0: Flush every second (data loss possible)
-- 1: Flush on every commit (ACID compliant, default)
-- 2: Write to OS cache on commit, flush every second

-- Performance vs Durability trade-off
SET GLOBAL innodb_flush_log_at_trx_commit = 1;  -- Production
SET GLOBAL innodb_flush_log_at_trx_commit = 2;  -- Replication slave
```

---

## 6. Undo Log

### Undo Log Purpose

- Transaction rollback
- MVCC (consistent read)
- Crash recovery

```
┌─────────────────────────────────────────────────────────────────┐
│                      Undo Log Structure                          │
│                                                                  │
│   Transaction Updates Row:                                       │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Data Page                                                │  │
│   │  ┌───────────────────────────────────────────────────┐   │  │
│   │  │ Row: id=1, name='New', roll_ptr ──────────────┐   │   │  │
│   │  └───────────────────────────────────────────────│───┘   │  │
│   └──────────────────────────────────────────────────│───────┘  │
│                                                      │           │
│                                                      ▼           │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Undo Segment                                             │  │
│   │  ┌────────────────────────────────────────────────────┐  │  │
│   │  │ Undo Record: name='Old', trx_id=100, roll_ptr ─┐   │  │  │
│   │  └──────────────────────────────────────────────│─┘   │  │  │
│   │                                                 │       │  │  │
│   │                                                 ▼       │  │  │
│   │  ┌────────────────────────────────────────────────────┐  │  │
│   │  │ Undo Record: name='Older', trx_id=50, roll_ptr=NULL│  │  │
│   │  └────────────────────────────────────────────────────┘  │  │
│   └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Undo Tablespace Configuration

```sql
-- View undo settings
SHOW VARIABLES LIKE 'innodb_undo%';

-- Configuration (MySQL 8.0+)
[mysqld]
innodb_undo_tablespaces = 2      -- Number of undo tablespaces
innodb_undo_log_truncate = ON    -- Enable truncation
innodb_max_undo_log_size = 1G    -- Truncation threshold
innodb_purge_rseg_truncate_frequency = 128

-- Monitor undo usage
SELECT
    TABLESPACE_NAME,
    FILE_NAME,
    TOTAL_EXTENTS,
    ALLOCATED_SIZE/1024/1024 AS allocated_mb
FROM information_schema.FILES
WHERE TABLESPACE_NAME LIKE 'innodb_undo%';
```

---

## 7. Change Buffer

Caches changes to secondary index pages when they're not in the buffer pool.

```
┌──────────────────────────────────────────────────────────────┐
│                     Change Buffer                             │
│                                                               │
│  INSERT into table with secondary index                       │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  1. Check if index page is in buffer pool                │ │
│  │     ├── YES → Update page directly                       │ │
│  │     └── NO  → Buffer change in change buffer             │ │
│  │                                                          │ │
│  │  2. Later, when page is read:                            │ │
│  │     └── Merge buffered changes into page                 │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  Supported operations:                                        │
│  - INSERT                                                     │
│  - DELETE-marking                                             │
│  - Purge (DELETE cleanup)                                     │
└──────────────────────────────────────────────────────────────┘
```

```sql
-- Change buffer settings
SHOW VARIABLES LIKE 'innodb_change_buffer%';

-- Configuration
innodb_change_buffering = all       -- inserts, deletes, purges, changes, all, none
innodb_change_buffer_max_size = 25  -- % of buffer pool (default 25%)

-- Monitor change buffer
SELECT
    NAME,
    COMMENT,
    SUBSYSTEM
FROM information_schema.INNODB_METRICS
WHERE NAME LIKE 'ibuf%';
```

---

## 8. Doublewrite Buffer

Protects against partial page writes during crash.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Doublewrite Process                           │
│                                                                  │
│  1. Flush dirty pages                                            │
│     Buffer Pool                                                  │
│     ┌────────────────────────────────────────────────────────┐  │
│     │ [Dirty Page 1] [Dirty Page 2] [Dirty Page 3]           │  │
│     └───────────────────────┬────────────────────────────────┘  │
│                             │                                    │
│  2. First Write: Sequential write to doublewrite buffer          │
│                             │                                    │
│                             ▼                                    │
│     ┌────────────────────────────────────────────────────────┐  │
│     │ Doublewrite Buffer (2 extents = 2MB)                   │  │
│     │ [Page 1 copy] [Page 2 copy] [Page 3 copy]              │  │
│     └───────────────────────┬────────────────────────────────┘  │
│                             │ fsync()                            │
│                             │                                    │
│  3. Second Write: Random write to actual locations               │
│                             │                                    │
│                             ▼                                    │
│     ┌────────────────────────────────────────────────────────┐  │
│     │ Tablespace Files (.ibd)                                │  │
│     │ [Page 1] ... [Page 2] ... [Page 3]                     │  │
│     └────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Crash Recovery: If tablespace page is corrupt,                  │
│                  restore from doublewrite buffer                 │
└─────────────────────────────────────────────────────────────────┘
```

```sql
-- Doublewrite settings
SHOW VARIABLES LIKE 'innodb_doublewrite%';

-- Configuration
innodb_doublewrite = ON              -- Enable (default)
innodb_doublewrite_dir = /fast/ssd   -- Location (8.0.20+)
innodb_doublewrite_pages = 128       -- Pages per batch

-- Disable for atomic write-capable storage
-- (some SSDs with battery backup, ZFS, etc.)
innodb_doublewrite = OFF
```

---

## 9. InnoDB Row Format

```sql
-- Available row formats
-- REDUNDANT: Legacy format
-- COMPACT: Default before 5.7
-- DYNAMIC: Default since 5.7, better for large columns
-- COMPRESSED: Data compression

-- Check table row format
SELECT TABLE_NAME, ROW_FORMAT
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb';

-- Set row format
CREATE TABLE example (
    id INT PRIMARY KEY,
    data TEXT
) ROW_FORMAT = DYNAMIC;

ALTER TABLE example ROW_FORMAT = COMPRESSED;
```

### Row Format Comparison

| Format | Off-page Storage | Compression | Best For |
|--------|-----------------|-------------|----------|
| REDUNDANT | 768 bytes prefix | No | Legacy |
| COMPACT | 768 bytes prefix | No | Small rows |
| DYNAMIC | 20-byte pointer only | No | Large columns |
| COMPRESSED | 20-byte pointer only | Yes | Storage-constrained |

---

## 10. InnoDB Monitoring

### Key Metrics

```sql
-- InnoDB engine status
SHOW ENGINE INNODB STATUS\G

-- Important sections:
-- BUFFER POOL AND MEMORY
-- ROW OPERATIONS
-- TRANSACTIONS
-- FILE I/O
-- LOG

-- Real-time monitoring via performance_schema
SELECT * FROM performance_schema.data_locks;
SELECT * FROM performance_schema.data_lock_waits;

-- InnoDB metrics
SELECT
    NAME,
    COUNT,
    AVG_COUNT,
    STATUS
FROM information_schema.INNODB_METRICS
WHERE NAME IN (
    'buffer_pool_size',
    'buffer_pool_reads',
    'buffer_pool_read_requests',
    'buffer_pool_write_requests',
    'log_writes',
    'trx_commits_insert_update'
);
```

### Health Check Query

```sql
SELECT
    'Buffer Pool Hit Ratio' AS metric,
    ROUND((1 - (
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') /
        NULLIF((SELECT VARIABLE_VALUE FROM performance_schema.global_status
                WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests'), 0)
    )) * 100, 2) AS value,
    '%' AS unit
UNION ALL
SELECT
    'Log Writes per Second',
    ROUND((SELECT VARIABLE_VALUE FROM performance_schema.global_status
           WHERE VARIABLE_NAME = 'Innodb_log_writes') /
          NULLIF((SELECT VARIABLE_VALUE FROM performance_schema.global_status
                  WHERE VARIABLE_NAME = 'Uptime'), 0), 2),
    'writes/sec';
```

---

## Summary

| Component | Purpose | Key Setting |
|-----------|---------|-------------|
| Buffer Pool | Cache data/index pages | `innodb_buffer_pool_size` |
| Redo Log | Crash recovery, durability | `innodb_log_file_size` |
| Undo Log | Rollback, MVCC | `innodb_undo_tablespaces` |
| Change Buffer | Buffer secondary index changes | `innodb_change_buffer_max_size` |
| Doublewrite | Prevent partial page writes | `innodb_doublewrite` |

---

## Further Reading

- InnoDB Storage Engine documentation
- "High Performance MySQL" - Buffer Pool tuning
- MySQL Server Blog - InnoDB internals
