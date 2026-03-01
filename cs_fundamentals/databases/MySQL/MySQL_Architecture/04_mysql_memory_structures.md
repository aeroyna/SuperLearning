# MySQL Memory Structures

## Learning Objectives
- Understand MySQL's memory allocation model
- Master buffer pool configuration and tuning
- Learn about various cache types
- Optimize memory usage for different workloads

---

## 1. Memory Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MySQL Memory Architecture                           │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                    GLOBAL BUFFERS (Shared)                          │ │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────┐   │ │
│  │  │ InnoDB Buffer    │  │ InnoDB Log       │  │ Key Buffer      │   │ │
│  │  │ Pool             │  │ Buffer           │  │ (MyISAM)        │   │ │
│  │  │ (12GB example)   │  │ (64MB)           │  │ (512MB)         │   │ │
│  │  └──────────────────┘  └──────────────────┘  └─────────────────┘   │ │
│  │                                                                     │ │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────┐   │ │
│  │  │ Table Open       │  │ Table Definition │  │ Binary Log      │   │ │
│  │  │ Cache            │  │ Cache            │  │ Cache           │   │ │
│  │  └──────────────────┘  └──────────────────┘  └─────────────────┘   │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                   PER-CONNECTION BUFFERS                            │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │ │
│  │  │ Thread 1    │  │ Thread 2    │  │ Thread 3    │  │ Thread N   │ │ │
│  │  │┌───────────┐│  │┌───────────┐│  │┌───────────┐│  │┌──────────┐│ │ │
│  │  ││sort_buffer││  ││sort_buffer││  ││sort_buffer││  ││sort_buff ││ │ │
│  │  ││join_buffer││  ││join_buffer││  ││join_buffer││  ││join_buff ││ │ │
│  │  ││read_buffer││  ││read_buffer││  ││read_buffer││  ││read_buff ││ │ │
│  │  │└───────────┘│  │└───────────┘│  │└───────────┘│  │└──────────┘│ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘ │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Global Buffers

### InnoDB Buffer Pool

The most important memory structure for InnoDB.

```sql
-- View buffer pool settings
SHOW VARIABLES LIKE 'innodb_buffer_pool%';

-- Key settings
innodb_buffer_pool_size = 12G       -- 70-80% of RAM for dedicated DB server
innodb_buffer_pool_instances = 8    -- 1 per GB, max 64
innodb_buffer_pool_chunk_size = 128M

-- Online resizing (MySQL 5.7+)
SET GLOBAL innodb_buffer_pool_size = 16G;

-- Monitor resize progress
SHOW STATUS LIKE 'Innodb_buffer_pool_resize_status';
```

#### Buffer Pool Sizing Formula

```
Total buffer pool = chunk_size × instances × n

Example:
innodb_buffer_pool_size = 12G
innodb_buffer_pool_instances = 8
innodb_buffer_pool_chunk_size = 128M

Each instance = 12G / 8 = 1.5G
Each instance has = 1.5G / 128M = 12 chunks
```

#### Buffer Pool Monitoring

```sql
-- Detailed buffer pool stats
SELECT
    POOL_ID,
    POOL_SIZE,
    FREE_BUFFERS,
    DATABASE_PAGES,
    OLD_DATABASE_PAGES,
    MODIFIED_DB_PAGES,
    PENDING_DECOMPRESS,
    PENDING_READS,
    PENDING_FLUSH_LRU,
    PENDING_FLUSH_LIST,
    PAGES_MADE_YOUNG,
    PAGES_NOT_MADE_YOUNG,
    PAGES_MADE_YOUNG_RATE,
    PAGES_MADE_NOT_YOUNG_RATE,
    NUMBER_PAGES_READ,
    NUMBER_PAGES_CREATED,
    NUMBER_PAGES_WRITTEN,
    HIT_RATE,
    YOUNG_MAKE_PER_THOUSAND_GETS,
    NOT_YOUNG_MAKE_PER_THOUSAND_GETS,
    NUMBER_PAGES_READ_AHEAD,
    NUMBER_READ_AHEAD_EVICTED
FROM information_schema.INNODB_BUFFER_POOL_STATS;

-- Hit ratio check
SELECT
    (1 - (
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') /
        NULLIF((SELECT VARIABLE_VALUE FROM performance_schema.global_status
                WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests'), 0)
    )) * 100 AS hit_ratio_percent;

-- Should be > 99% for healthy system
```

### InnoDB Log Buffer

Caches redo log records before writing to disk.

```sql
-- Log buffer settings
SHOW VARIABLES LIKE 'innodb_log_buffer_size';

-- Default: 16MB, increase for large transactions
innodb_log_buffer_size = 64M

-- Flush behavior
innodb_flush_log_at_trx_commit = 1  -- Flush on every commit (ACID)
innodb_flush_log_at_trx_commit = 2  -- Write to OS cache on commit
innodb_flush_log_at_trx_commit = 0  -- Flush every second (fastest, least safe)
```

### Key Buffer (MyISAM)

```sql
-- Key buffer for MyISAM indexes only
SHOW VARIABLES LIKE 'key_buffer_size';

-- Size appropriately for MyISAM workloads
key_buffer_size = 512M

-- Multiple key buffers (advanced)
-- Create named key buffer
SET GLOBAL hot_cache.key_buffer_size = 256M;

-- Assign tables to specific cache
CACHE INDEX my_table IN hot_cache;
LOAD INDEX INTO CACHE my_table;
```

### Table Open Cache

```sql
-- Number of open table handles
SHOW VARIABLES LIKE 'table_open_cache';
SHOW VARIABLES LIKE 'table_open_cache_instances';

-- Default varies by version
table_open_cache = 4000
table_open_cache_instances = 16  -- Reduce contention

-- Monitor usage
SHOW GLOBAL STATUS LIKE 'Open%tables';
SHOW GLOBAL STATUS LIKE 'Opened_tables';

-- If Opened_tables grows rapidly, increase cache
-- Opened_tables / Uptime < 1 per second is good
```

### Table Definition Cache

```sql
-- Caches .frm file parsing (pre-8.0) / data dictionary (8.0+)
SHOW VARIABLES LIKE 'table_definition_cache';

-- Typically matches table_open_cache
table_definition_cache = 4000

-- Monitor
SHOW GLOBAL STATUS LIKE 'Open_table_definitions';
SHOW GLOBAL STATUS LIKE 'Opened_table_definitions';
```

---

## 3. Per-Connection Buffers

### Sort Buffer

```sql
-- Used for ORDER BY, GROUP BY without index
SHOW VARIABLES LIKE 'sort_buffer_size';

-- Default: 256KB - 2MB
sort_buffer_size = 2M

-- Allocated per connection when sorting
-- Don't set too high: max_connections × sort_buffer_size
-- 500 connections × 4MB = 2GB potential allocation!

-- Monitor sort activity
SHOW GLOBAL STATUS LIKE 'Sort%';
/*
Sort_merge_passes - Indicates disk sorts (increase buffer if high)
Sort_range
Sort_rows
Sort_scan
*/
```

### Join Buffer

```sql
-- Used for joins without indexes (block nested loop)
SHOW VARIABLES LIKE 'join_buffer_size';

-- Default: 256KB
join_buffer_size = 2M

-- Multiple buffers can be used per join
-- MySQL 8.0.18+ uses hash join which benefits from larger buffer

-- Monitor
EXPLAIN FORMAT=JSON SELECT ... -- Check for "Using join buffer"
```

### Read Buffers

```sql
-- Sequential scan buffer
SHOW VARIABLES LIKE 'read_buffer_size';
read_buffer_size = 256K  -- Default 128K

-- Random read buffer (for sorted reads)
SHOW VARIABLES LIKE 'read_rnd_buffer_size';
read_rnd_buffer_size = 512K  -- Default 256K

-- These are allocated per-table in a query
-- Multiple tables = multiple buffers
```

### Other Per-Connection Buffers

```sql
-- Network buffer
SHOW VARIABLES LIKE 'net_buffer_length';
SHOW VARIABLES LIKE 'max_allowed_packet';

net_buffer_length = 16K
max_allowed_packet = 64M

-- Bulk insert buffer (for bulk loads)
SHOW VARIABLES LIKE 'bulk_insert_buffer_size';
bulk_insert_buffer_size = 8M

-- Temporary table buffer
SHOW VARIABLES LIKE 'tmp_table_size';
SHOW VARIABLES LIKE 'max_heap_table_size';

tmp_table_size = 64M
max_heap_table_size = 64M  -- Must match tmp_table_size
```

---

## 4. Thread Memory

### Thread Stack

```sql
-- Stack size per thread
SHOW VARIABLES LIKE 'thread_stack';
thread_stack = 256K  -- Default varies

-- Rarely needs adjustment unless:
-- - Complex stored procedures
-- - Deep recursion
-- - Stack overflow errors
```

### Thread Cache

```sql
-- Cache threads for connection reuse
SHOW VARIABLES LIKE 'thread_cache_size';
thread_cache_size = 100

-- Monitor thread creation
SHOW GLOBAL STATUS LIKE 'Threads_created';
SHOW GLOBAL STATUS LIKE 'Threads_cached';
SHOW GLOBAL STATUS LIKE 'Connections';

-- Thread creation rate
-- Threads_created / Connections should be low
```

---

## 5. Query Cache (Deprecated)

```sql
-- REMOVED in MySQL 8.0
-- Still exists in 5.7 and earlier

-- Check if enabled
SHOW VARIABLES LIKE 'query_cache%';

-- Configuration (5.7)
query_cache_type = 0      -- 0=OFF, 1=ON, 2=DEMAND
query_cache_size = 0      -- Set to 0 to disable

-- Why removed:
-- 1. Single mutex caused contention
-- 2. Invalidation overhead
-- 3. Memory fragmentation
-- 4. Better alternatives (ProxySQL, Redis)
```

---

## 6. Memory Calculation

### Estimate Total Memory Usage

```sql
-- Formula for maximum memory usage:

-- Global buffers (fixed)
-- innodb_buffer_pool_size
-- + innodb_log_buffer_size
-- + key_buffer_size
-- + innodb_additional_mem_pool_size (deprecated)

-- Per-connection (variable)
-- + max_connections × (
--     sort_buffer_size
--     + join_buffer_size
--     + read_buffer_size
--     + read_rnd_buffer_size
--     + thread_stack
--     + binlog_cache_size (if binlog enabled)
--   )

-- Example calculation:
SELECT
    (@@innodb_buffer_pool_size +
     @@innodb_log_buffer_size +
     @@key_buffer_size) / 1024 / 1024 / 1024 AS global_buffers_gb,
    @@max_connections AS max_connections,
    (@@sort_buffer_size +
     @@join_buffer_size +
     @@read_buffer_size +
     @@read_rnd_buffer_size +
     @@thread_stack) / 1024 / 1024 AS per_connection_mb,
    ((@@sort_buffer_size +
      @@join_buffer_size +
      @@read_buffer_size +
      @@read_rnd_buffer_size +
      @@thread_stack) * @@max_connections) / 1024 / 1024 / 1024 AS max_connection_memory_gb;
```

### Memory Sizing Script

```sql
-- Comprehensive memory analysis
SELECT
    '=== Global Buffers ===' AS category, '' AS value
UNION ALL
SELECT 'innodb_buffer_pool_size',
    CONCAT(@@innodb_buffer_pool_size/1024/1024/1024, ' GB')
UNION ALL
SELECT 'innodb_log_buffer_size',
    CONCAT(@@innodb_log_buffer_size/1024/1024, ' MB')
UNION ALL
SELECT 'key_buffer_size',
    CONCAT(@@key_buffer_size/1024/1024, ' MB')
UNION ALL
SELECT '', ''
UNION ALL
SELECT '=== Per-Connection Buffers ===' AS category, '' AS value
UNION ALL
SELECT 'sort_buffer_size',
    CONCAT(@@sort_buffer_size/1024, ' KB')
UNION ALL
SELECT 'join_buffer_size',
    CONCAT(@@join_buffer_size/1024, ' KB')
UNION ALL
SELECT 'read_buffer_size',
    CONCAT(@@read_buffer_size/1024, ' KB')
UNION ALL
SELECT 'read_rnd_buffer_size',
    CONCAT(@@read_rnd_buffer_size/1024, ' KB')
UNION ALL
SELECT '', ''
UNION ALL
SELECT '=== Limits ===' AS category, '' AS value
UNION ALL
SELECT 'max_connections', @@max_connections
UNION ALL
SELECT 'tmp_table_size',
    CONCAT(@@tmp_table_size/1024/1024, ' MB');
```

---

## 7. Memory Optimization Strategies

### For OLTP Workloads

```ini
[mysqld]
# Large buffer pool, smaller per-connection
innodb_buffer_pool_size = 12G
innodb_buffer_pool_instances = 8

# Moderate per-connection (many concurrent)
sort_buffer_size = 256K
join_buffer_size = 256K
read_buffer_size = 128K
read_rnd_buffer_size = 256K

# Many connections
max_connections = 500
thread_cache_size = 100
```

### For OLAP/Reporting Workloads

```ini
[mysqld]
# Still large buffer pool
innodb_buffer_pool_size = 12G

# Larger per-connection for complex queries
sort_buffer_size = 4M
join_buffer_size = 4M
read_buffer_size = 1M
read_rnd_buffer_size = 2M

# Larger temp tables
tmp_table_size = 256M
max_heap_table_size = 256M

# Fewer connections
max_connections = 50
```

### Memory Monitoring

```sql
-- Current memory usage (Performance Schema)
SELECT
    EVENT_NAME,
    CURRENT_NUMBER_OF_BYTES_USED/1024/1024 AS mb_used
FROM performance_schema.memory_summary_global_by_event_name
WHERE CURRENT_NUMBER_OF_BYTES_USED > 0
ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC
LIMIT 20;

-- Memory by user
SELECT
    USER,
    HOST,
    CURRENT_NUMBER_OF_BYTES_USED/1024/1024 AS mb_used
FROM performance_schema.memory_summary_by_account_by_event_name
WHERE EVENT_NAME = 'memory/sql/THD::main_mem_root'
ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC
LIMIT 10;

-- Memory instruments (enable first)
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES'
WHERE NAME LIKE 'memory/%';
```

---

## 8. Troubleshooting Memory Issues

### Out of Memory Errors

```bash
# Check MySQL error log
tail -f /var/log/mysql/error.log

# Common messages:
# [ERROR] Out of memory
# [ERROR] Cannot allocate memory for the buffer pool

# Solutions:
# 1. Reduce innodb_buffer_pool_size
# 2. Reduce max_connections
# 3. Reduce per-connection buffers
# 4. Add swap (not recommended for production)
# 5. Add physical RAM
```

### Memory Leak Detection

```sql
-- Track memory growth over time
SELECT
    EVENT_NAME,
    CURRENT_NUMBER_OF_BYTES_USED,
    HIGH_NUMBER_OF_BYTES_USED
FROM performance_schema.memory_summary_global_by_event_name
WHERE HIGH_NUMBER_OF_BYTES_USED > CURRENT_NUMBER_OF_BYTES_USED * 1.5
ORDER BY HIGH_NUMBER_OF_BYTES_USED DESC;

-- Check for runaway connections
SELECT
    USER,
    COUNT(*) AS connections,
    SUM(CURRENT_NUMBER_OF_BYTES_USED)/1024/1024 AS total_mb
FROM performance_schema.memory_summary_by_account_by_event_name m
JOIN performance_schema.accounts a USING (USER, HOST)
GROUP BY USER
ORDER BY total_mb DESC;
```

### Swap Usage

```bash
# Check swap usage
free -h
vmstat 1 5

# Check MySQL swap specifically
pid=$(pgrep -x mysqld)
grep Swap /proc/$pid/smaps | awk '{sum+=$2} END {print sum/1024 " MB"}'

# If MySQL is swapping significantly:
# 1. Reduce memory usage
# 2. Tune vm.swappiness
echo 10 > /proc/sys/vm/swappiness
```

---

## 9. Configuration Template

### 16GB RAM Server

```ini
[mysqld]
# InnoDB Buffer Pool (70% of RAM)
innodb_buffer_pool_size = 11G
innodb_buffer_pool_instances = 8
innodb_buffer_pool_chunk_size = 128M

# InnoDB Log
innodb_log_buffer_size = 64M
innodb_log_file_size = 1G

# Per-connection
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 256K
read_rnd_buffer_size = 512K

# Temp tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Connections
max_connections = 200
thread_cache_size = 50

# Table cache
table_open_cache = 4000
table_definition_cache = 4000
table_open_cache_instances = 16
```

---

## Summary

| Buffer Type | Scope | Key Setting | Typical Size |
|------------|-------|-------------|--------------|
| Buffer Pool | Global | innodb_buffer_pool_size | 70-80% RAM |
| Log Buffer | Global | innodb_log_buffer_size | 16-64MB |
| Key Buffer | Global | key_buffer_size | Based on MyISAM usage |
| Sort Buffer | Per-connection | sort_buffer_size | 256K-4M |
| Join Buffer | Per-connection | join_buffer_size | 256K-4M |
| Table Cache | Global | table_open_cache | 4000+ |

---

## Further Reading

- MySQL Memory Allocation documentation
- "High Performance MySQL" - Memory optimization
- MySQL Performance Blog - Memory tuning
