# PostgreSQL Memory Architecture

## Learning Objectives
- Understand shared memory components
- Master buffer management and caching
- Configure memory for optimal performance
- Monitor memory usage effectively

---

## 1. Memory Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Memory Architecture                        │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                      SHARED MEMORY                                  │ │
│  │                  (Accessible by all processes)                      │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │ Shared Buffers                                      (128MB+) │  │ │
│  │  │ • Data page cache                                            │  │ │
│  │  │ • Index page cache                                           │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │ WAL Buffers                                          (16MB)  │  │ │
│  │  │ • Write-ahead log buffer                                     │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  │                                                                     │ │
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────┐  │ │
│  │  │ Lock Tables     │ │ Proc Array      │ │ Other Structures    │  │ │
│  │  │ • Lock info     │ │ • Process list  │ │ • CLOG buffers      │  │ │
│  │  │ • Lock queues   │ │ • Transaction   │ │ • Subtrans buffers  │  │ │
│  │  └─────────────────┘ └─────────────────┘ └─────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                     PER-BACKEND MEMORY                              │ │
│  │                  (Private to each process)                          │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │ work_mem           (4MB default)                              │  │ │
│  │  │ • Sorting, hash tables, hash joins                           │  │ │
│  │  │ • Per-operation (not per-query!)                             │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │ maintenance_work_mem  (64MB default)                          │  │ │
│  │  │ • VACUUM, CREATE INDEX, ALTER TABLE                          │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │ temp_buffers         (8MB default)                            │  │ │
│  │  │ • Temporary table access                                      │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Shared Buffers

### Purpose and Configuration

```sql
-- View shared_buffers setting
SHOW shared_buffers;

-- Recommended: 25% of RAM for dedicated server
-- Example: 32GB RAM → 8GB shared_buffers
-- Example: 64GB RAM → 16GB shared_buffers

-- postgresql.conf
-- shared_buffers = 8GB
```

### Buffer Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Shared Buffers Structure                          │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Buffer Descriptors                        │    │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                    │    │
│  │  │ BD0 │ │ BD1 │ │ BD2 │ │ BD3 │ │ ... │                    │    │
│  │  │     │ │     │ │     │ │     │ │     │                    │    │
│  │  │tag  │ │tag  │ │tag  │ │tag  │ │tag  │  (metadata)        │    │
│  │  │flags│ │flags│ │flags│ │flags│ │flags│                    │    │
│  │  │ref  │ │ref  │ │ref  │ │ref  │ │ref  │                    │    │
│  │  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘                    │    │
│  └─────┼───────┼───────┼───────┼───────┼───────────────────────┘    │
│        │       │       │       │       │                             │
│        ▼       ▼       ▼       ▼       ▼                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Buffer Pool                               │    │
│  │  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐          │    │
│  │  │ Page 0│ │ Page 1│ │ Page 2│ │ Page 3│ │  ...  │          │    │
│  │  │ (8KB) │ │ (8KB) │ │ (8KB) │ │ (8KB) │ │ (8KB) │          │    │
│  │  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Buffer Tag: (tablespace, database, relation, fork, block#)         │
│  Flags: dirty, valid, locked, io_in_progress, etc.                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Buffer Lookup

```sql
-- Check buffer cache contents
SELECT
    c.relname,
    count(*) AS buffers,
    pg_size_pretty(count(*) * 8192) AS buffer_size
FROM pg_buffercache b
JOIN pg_class c ON b.relfilenode = c.relfilenode
WHERE b.reldatabase = (SELECT oid FROM pg_database WHERE datname = current_database())
GROUP BY c.relname
ORDER BY count(*) DESC
LIMIT 20;

-- Buffer hit ratio
SELECT
    sum(heap_blks_hit) AS hit,
    sum(heap_blks_read) AS read,
    sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit + heap_blks_read), 0) AS ratio
FROM pg_statio_user_tables;
```

### Clock Sweep Algorithm

PostgreSQL uses a clock sweep (variant of LRU) for buffer replacement:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Clock Sweep Algorithm                             │
│                                                                      │
│              ┌───────────────────────┐                               │
│              │    Usage Counter      │                               │
│              │    (0-5 typically)    │                               │
│              └───────────────────────┘                               │
│                                                                      │
│  Buffer Ring:                                                        │
│                      Clock Hand                                      │
│                         ↓                                            │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                   │
│  │  0  │ │  3  │ │  1  │ │  0  │ │  2  │ │  0  │                   │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                   │
│    ↓                                              ↓                  │
│  Evict                                        Evict                  │
│  candidate                                    candidate              │
│                                                                      │
│  Rules:                                                              │
│  1. When accessed, usage_count++ (max BM_MAX_USAGE_COUNT)           │
│  2. Clock hand decrements usage_count                                │
│  3. When usage_count = 0, buffer can be evicted                      │
│  4. Dirty buffers written before eviction                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. WAL Buffers

```sql
-- View WAL buffer settings
SHOW wal_buffers;

-- Default: 1/32 of shared_buffers, max 16MB
-- Usually auto-tuned: -1 means auto

-- postgresql.conf
-- wal_buffers = 64MB  -- Or -1 for auto
```

### WAL Buffer Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WAL Buffer Flow                                   │
│                                                                      │
│  Backend Process                                                     │
│       │                                                              │
│       │ Write WAL record                                             │
│       ▼                                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    WAL Buffers                               │    │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                    │    │
│  │  │ rec │ │ rec │ │ rec │ │     │ │     │                    │    │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                    │    │
│  │        ↑                                                     │    │
│  │     Insert                                                   │    │
│  │     Position                                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                         │                                            │
│                         │ WAL Writer or Commit                       │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    WAL Files (pg_wal/)                       │    │
│  │  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐   │    │
│  │  │ 000000010000... │ │ 000000010000... │ │ 000000010000... │   │    │
│  │  └────────────────┘ └────────────────┘ └────────────────┘   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Sync settings:                                                      │
│  • synchronous_commit = on (wait for disk write)                     │
│  • synchronous_commit = off (don't wait, faster)                     │
│  • fsync = on (actual disk sync)                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Per-Backend Memory

### work_mem

```sql
-- View current setting
SHOW work_mem;

-- Set for session
SET work_mem = '256MB';

-- Important: Allocated per-operation, not per-query!
-- A single query with multiple sorts/hashes can use multiples of work_mem

-- Example: Query with 5 sort operations
-- Memory used = 5 × work_mem = 5 × 256MB = 1.28GB
```

```sql
-- Check if operations spill to disk
EXPLAIN ANALYZE SELECT * FROM large_table ORDER BY column1;

-- Look for:
-- "Sort Method: external merge  Disk: 12345kB"  -- Bad, increase work_mem
-- "Sort Method: quicksort  Memory: 1234kB"      -- Good, fits in memory
```

### maintenance_work_mem

```sql
-- View setting
SHOW maintenance_work_mem;

-- Used for:
-- • VACUUM
-- • CREATE INDEX
-- • ALTER TABLE ADD FOREIGN KEY

-- Can be set higher than work_mem since these are single operations
-- postgresql.conf
-- maintenance_work_mem = 1GB

-- Or set for specific session
SET maintenance_work_mem = '2GB';
CREATE INDEX CONCURRENTLY idx_large ON large_table(column1);
RESET maintenance_work_mem;
```

### temp_buffers

```sql
-- For temporary tables
SHOW temp_buffers;

-- Must be set before first use of temp tables in session
SET temp_buffers = '128MB';

-- Then create temp table
CREATE TEMP TABLE temp_data AS SELECT * FROM source_table;
```

---

## 5. Memory Configuration

### Sizing Guidelines

```ini
# postgresql.conf for 32GB RAM server

# Shared Memory
shared_buffers = 8GB                    # 25% of RAM
effective_cache_size = 24GB             # 75% of RAM (estimate for OS cache)
wal_buffers = 64MB                      # Or -1 for auto

# Per-Backend Memory
work_mem = 64MB                         # Careful with max_connections
maintenance_work_mem = 1GB              # For maintenance operations
temp_buffers = 32MB                     # For temp tables

# Huge Pages (Linux)
huge_pages = try                        # or 'on' if configured

# Hash operations
hash_mem_multiplier = 2.0               # PostgreSQL 13+
```

### Memory Calculation

```sql
-- Estimate maximum memory usage

-- Shared memory (fixed):
-- shared_buffers + wal_buffers + fixed overhead

-- Per-connection (variable):
-- work_mem × max_connections × average_operations_per_query

-- Worst case formula:
-- Total = shared_buffers + (max_connections × work_mem × estimated_operations)

-- Example:
-- shared_buffers = 8GB
-- max_connections = 200
-- work_mem = 64MB
-- Estimated operations = 3

-- Worst case: 8GB + (200 × 64MB × 3) = 8GB + 38.4GB = 46.4GB
-- This exceeds 32GB RAM! Either reduce work_mem or max_connections
```

### Huge Pages

```bash
# Calculate huge pages needed
# shared_buffers / 2MB (huge page size)

# For 8GB shared_buffers:
# 8GB / 2MB = 4096 huge pages

# Set in /etc/sysctl.conf
vm.nr_hugepages = 4200  # A bit more for overhead

# Apply
sysctl -p

# postgresql.conf
huge_pages = on
```

---

## 6. Monitoring Memory

### Shared Buffer Statistics

```sql
-- pg_buffercache extension
CREATE EXTENSION pg_buffercache;

-- Buffer usage summary
SELECT
    CASE
        WHEN usagecount IS NULL THEN 'unused'
        ELSE usagecount::text
    END AS usage,
    count(*)
FROM pg_buffercache
GROUP BY usagecount
ORDER BY usagecount NULLS FIRST;

-- Dirty buffer count
SELECT count(*) AS dirty_buffers
FROM pg_buffercache
WHERE isdirty;
```

### Cache Hit Ratios

```sql
-- Table cache hit ratio
SELECT
    schemaname,
    relname,
    heap_blks_read,
    heap_blks_hit,
    ROUND(heap_blks_hit::numeric /
        NULLIF(heap_blks_hit + heap_blks_read, 0) * 100, 2) AS hit_ratio
FROM pg_statio_user_tables
WHERE heap_blks_read > 0
ORDER BY heap_blks_read DESC
LIMIT 10;

-- Index cache hit ratio
SELECT
    schemaname,
    relname,
    indexrelname,
    idx_blks_read,
    idx_blks_hit,
    ROUND(idx_blks_hit::numeric /
        NULLIF(idx_blks_hit + idx_blks_read, 0) * 100, 2) AS hit_ratio
FROM pg_statio_user_indexes
WHERE idx_blks_read > 0
ORDER BY idx_blks_read DESC
LIMIT 10;
```

### Memory Contexts (PostgreSQL 14+)

```sql
-- View backend memory contexts
SELECT * FROM pg_backend_memory_contexts;

-- Memory by context type
SELECT
    name,
    pg_size_pretty(total_bytes) AS total,
    pg_size_pretty(used_bytes) AS used,
    pg_size_pretty(free_bytes) AS free
FROM pg_backend_memory_contexts
ORDER BY total_bytes DESC
LIMIT 20;
```

---

## 7. Operating System Considerations

### Linux Memory Settings

```bash
# /etc/sysctl.conf

# Prevent OOM killer from targeting PostgreSQL
vm.overcommit_memory = 2
vm.overcommit_ratio = 80

# Swappiness (minimize swapping)
vm.swappiness = 1

# Dirty page ratio
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5
```

### Shared Memory Settings

```bash
# /etc/sysctl.conf

# Maximum shared memory segment (bytes)
kernel.shmmax = 17179869184  # 16GB

# Maximum total shared memory (pages)
kernel.shmall = 4194304      # 16GB / 4KB page size

# Apply
sysctl -p
```

---

## Summary

| Memory Area | Setting | Default | Recommendation |
|-------------|---------|---------|----------------|
| Shared Buffers | shared_buffers | 128MB | 25% of RAM |
| Effective Cache | effective_cache_size | 4GB | 50-75% of RAM |
| WAL Buffers | wal_buffers | -1 (auto) | 64MB for busy systems |
| Work Memory | work_mem | 4MB | 64-256MB (careful!) |
| Maintenance | maintenance_work_mem | 64MB | 1-2GB |
| Temp Buffers | temp_buffers | 8MB | As needed |

---

## Further Reading

- PostgreSQL Memory Configuration documentation
- "PostgreSQL 14 Internals" book
- PostgreSQL wiki on Tuning
