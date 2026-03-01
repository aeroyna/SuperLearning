# Performance Tuning

## Learning Objectives
- Configure MySQL for optimal performance
- Tune InnoDB settings for your workload
- Optimize operating system settings
- Establish performance baselines

---

## 1. Configuration Approach

### Tuning Methodology

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Performance Tuning Cycle                          │
│                                                                      │
│    ┌──────────────┐                                                  │
│    │  1. MEASURE  │ ← Establish baseline                             │
│    └──────┬───────┘                                                  │
│           │                                                          │
│           ▼                                                          │
│    ┌──────────────┐                                                  │
│    │  2. ANALYZE  │ ← Identify bottlenecks                           │
│    └──────┬───────┘                                                  │
│           │                                                          │
│           ▼                                                          │
│    ┌──────────────┐                                                  │
│    │   3. TUNE    │ ← Change ONE setting                             │
│    └──────┬───────┘                                                  │
│           │                                                          │
│           ▼                                                          │
│    ┌──────────────┐                                                  │
│    │  4. VERIFY   │ ← Measure impact                                 │
│    └──────┬───────┘                                                  │
│           │                                                          │
│           └────────────► Repeat                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Starting Point

```sql
-- System information
SHOW VARIABLES LIKE 'version';
SHOW VARIABLES LIKE 'innodb_version';

-- Current configuration
SHOW VARIABLES WHERE Variable_name IN (
    'innodb_buffer_pool_size',
    'innodb_log_file_size',
    'max_connections',
    'table_open_cache'
);

-- Current status
SHOW GLOBAL STATUS WHERE Variable_name IN (
    'Uptime',
    'Questions',
    'Threads_connected',
    'Innodb_buffer_pool_read_requests',
    'Innodb_buffer_pool_reads'
);
```

---

## 2. Memory Configuration

### InnoDB Buffer Pool

```ini
# my.cnf - Most important setting
[mysqld]
# 70-80% of available RAM for dedicated DB server
innodb_buffer_pool_size = 12G

# Multiple instances reduce contention (1 per GB, max 64)
innodb_buffer_pool_instances = 8

# Chunk size for online resizing
innodb_buffer_pool_chunk_size = 128M

# Dump and load buffer pool on restart
innodb_buffer_pool_dump_at_shutdown = ON
innodb_buffer_pool_load_at_startup = ON
```

### Calculating Buffer Pool Size

```sql
-- Check data size
SELECT
    ROUND(SUM(data_length + index_length) / 1024 / 1024 / 1024, 2) AS total_gb
FROM information_schema.TABLES;

-- Ideal: Buffer pool > total data size
-- Minimum: Buffer pool > frequently accessed data (working set)

-- Check buffer pool hit ratio
SELECT
    (1 - (
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') /
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests')
    )) * 100 AS hit_ratio;

-- Target: > 99%
```

### Other Memory Settings

```ini
[mysqld]
# InnoDB log buffer
innodb_log_buffer_size = 64M

# Per-connection buffers (be careful with max_connections!)
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 256K
read_rnd_buffer_size = 512K

# Temp tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Table cache
table_open_cache = 4000
table_definition_cache = 4000
table_open_cache_instances = 16
```

---

## 3. InnoDB Tuning

### Transaction Log Settings

```ini
[mysqld]
# Redo log size (total across all files)
innodb_redo_log_capacity = 2G  # MySQL 8.0.30+
# Or for older versions:
# innodb_log_file_size = 1G
# innodb_log_files_in_group = 2

# Durability settings
# 1 = Full ACID (safest, slower)
# 2 = Flush every second (faster, some risk)
# 0 = No sync (fastest, highest risk)
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
```

### I/O Settings

```ini
[mysqld]
# Flush method (Linux)
innodb_flush_method = O_DIRECT

# I/O capacity (adjust for your storage)
innodb_io_capacity = 200        # Standard HDD
# innodb_io_capacity = 2000     # SSD
# innodb_io_capacity = 10000    # High-end SSD/NVMe

innodb_io_capacity_max = 4000   # 2x io_capacity for bursts

# I/O threads
innodb_read_io_threads = 4
innodb_write_io_threads = 4

# Page cleaners (for dirty page flushing)
innodb_page_cleaners = 4
```

### Concurrency Settings

```ini
[mysqld]
# Let InnoDB manage threads (0 = auto)
innodb_thread_concurrency = 0

# Purge threads (undo log cleanup)
innodb_purge_threads = 4

# Adaptive flushing
innodb_adaptive_flushing = ON
innodb_adaptive_flushing_lwm = 10
```

---

## 4. Connection Settings

### Connection Limits

```ini
[mysqld]
# Maximum connections
max_connections = 500

# Connection timeout
wait_timeout = 28800          # 8 hours (idle non-interactive)
interactive_timeout = 28800    # 8 hours (interactive)
connect_timeout = 10           # Connection establishment timeout

# Thread cache
thread_cache_size = 100        # Reuse threads

# Max allowed packet
max_allowed_packet = 64M
```

### Connection Pool Sizing

```sql
-- Check connection usage
SHOW STATUS LIKE 'Max_used_connections';
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Connections';

-- Calculate appropriate max_connections
-- max_connections = (Available RAM - Buffer Pool - OS) / per_connection_memory
-- per_connection_memory ≈ 2-5MB typically

-- Check thread creation
SHOW STATUS LIKE 'Threads_created';
-- Should be low if thread_cache_size is adequate
```

---

## 5. Query Cache (MySQL 5.7 and earlier)

```ini
# DEPRECATED - Removed in MySQL 8.0
# Generally not recommended due to contention

[mysqld]
# Disable query cache (recommended)
query_cache_type = 0
query_cache_size = 0

# If you must use it:
# query_cache_type = 1
# query_cache_size = 128M
# query_cache_limit = 2M
```

---

## 6. Operating System Tuning

### Linux Settings

```bash
# /etc/sysctl.conf

# Virtual memory
vm.swappiness = 1                    # Minimize swapping
vm.dirty_ratio = 60
vm.dirty_background_ratio = 10

# Network
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.core.netdev_max_backlog = 65535

# File descriptors
fs.file-max = 65535
```

### Ulimits

```bash
# /etc/security/limits.conf
mysql    soft    nofile    65535
mysql    hard    nofile    65535
mysql    soft    nproc     65535
mysql    hard    nproc     65535
```

### Disk I/O Scheduler

```bash
# For SSDs
echo noop > /sys/block/sda/queue/scheduler
# Or
echo none > /sys/block/nvme0n1/queue/scheduler

# For HDDs
echo deadline > /sys/block/sda/queue/scheduler
```

### Filesystem

```bash
# Recommended mount options for MySQL data directory
# /etc/fstab
/dev/sda1  /var/lib/mysql  ext4  noatime,nobarrier  0  2

# XFS (often better for MySQL)
/dev/sda1  /var/lib/mysql  xfs   noatime,nodiratime  0  2
```

---

## 7. Workload-Specific Tuning

### OLTP Workload

```ini
[mysqld]
# High concurrency, many small transactions
innodb_buffer_pool_size = 12G
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
innodb_flush_method = O_DIRECT
max_connections = 500
innodb_io_capacity = 2000

# Small per-connection buffers
sort_buffer_size = 256K
join_buffer_size = 256K
```

### OLAP/Reporting Workload

```ini
[mysqld]
# Complex queries, large scans
innodb_buffer_pool_size = 12G
innodb_flush_log_at_trx_commit = 2  # Less critical for reads
max_connections = 50

# Larger per-connection buffers
sort_buffer_size = 4M
join_buffer_size = 4M
read_buffer_size = 1M
read_rnd_buffer_size = 2M
tmp_table_size = 256M
max_heap_table_size = 256M
```

### Mixed Workload

```ini
[mysqld]
# Balance between OLTP and OLAP
innodb_buffer_pool_size = 12G
innodb_flush_log_at_trx_commit = 1
max_connections = 200

# Moderate buffers
sort_buffer_size = 2M
join_buffer_size = 2M
tmp_table_size = 64M
max_heap_table_size = 64M
```

---

## 8. Configuration Templates

### 16GB RAM Server

```ini
[mysqld]
# InnoDB
innodb_buffer_pool_size = 11G
innodb_buffer_pool_instances = 8
innodb_log_buffer_size = 64M
innodb_redo_log_capacity = 2G
innodb_flush_log_at_trx_commit = 1
innodb_flush_method = O_DIRECT
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000

# Connections
max_connections = 200
thread_cache_size = 50
table_open_cache = 4000

# Per-connection
sort_buffer_size = 2M
join_buffer_size = 2M
read_buffer_size = 256K

# Temp tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Logging
slow_query_log = 1
long_query_time = 2
```

### 64GB RAM Server

```ini
[mysqld]
# InnoDB
innodb_buffer_pool_size = 48G
innodb_buffer_pool_instances = 48
innodb_log_buffer_size = 128M
innodb_redo_log_capacity = 4G
innodb_flush_log_at_trx_commit = 1
innodb_flush_method = O_DIRECT
innodb_io_capacity = 4000
innodb_io_capacity_max = 8000

# Connections
max_connections = 500
thread_cache_size = 100
table_open_cache = 8000

# Per-connection
sort_buffer_size = 4M
join_buffer_size = 4M
read_buffer_size = 512K

# Temp tables
tmp_table_size = 128M
max_heap_table_size = 128M
```

---

## 9. Verification and Monitoring

### Check Configuration Impact

```sql
-- Buffer pool efficiency
SELECT
    (1 - Innodb_buffer_pool_reads/Innodb_buffer_pool_read_requests) * 100 AS hit_ratio
FROM (
    SELECT
        MAX(CASE WHEN VARIABLE_NAME = 'Innodb_buffer_pool_reads'
            THEN VARIABLE_VALUE END) AS Innodb_buffer_pool_reads,
        MAX(CASE WHEN VARIABLE_NAME = 'Innodb_buffer_pool_read_requests'
            THEN VARIABLE_VALUE END) AS Innodb_buffer_pool_read_requests
    FROM performance_schema.global_status
) t;

-- Connection efficiency
SELECT
    (Threads_created / Connections) * 100 AS thread_creation_rate
FROM (
    SELECT
        MAX(CASE WHEN VARIABLE_NAME = 'Threads_created'
            THEN VARIABLE_VALUE END) AS Threads_created,
        MAX(CASE WHEN VARIABLE_NAME = 'Connections'
            THEN VARIABLE_VALUE END) AS Connections
    FROM performance_schema.global_status
) t;
```

---

## Summary

| Setting | Starting Point | Adjust Based On |
|---------|----------------|-----------------|
| innodb_buffer_pool_size | 70% RAM | Hit ratio |
| max_connections | 200 | Peak usage |
| innodb_io_capacity | 200-2000 | Storage type |
| sort_buffer_size | 256K-2M | Complex queries |

---

## Further Reading

- MySQL Server Configuration documentation
- "High Performance MySQL" - Configuration chapters
- Percona Configuration Wizard
