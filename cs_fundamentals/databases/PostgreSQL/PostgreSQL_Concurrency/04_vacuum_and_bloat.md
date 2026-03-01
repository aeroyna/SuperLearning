# Vacuum and Bloat Management

## Learning Objectives
- Understand why VACUUM is necessary in PostgreSQL
- Configure autovacuum for optimal performance
- Detect and remediate table bloat
- Choose between VACUUM and VACUUM FULL

---

## 1. Why VACUUM is Necessary

### MVCC Creates Dead Tuples

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Dead Tuple Accumulation                           │
│                                                                      │
│  UPDATE creates new tuple, leaves old tuple "dead":                  │
│                                                                      │
│  Before UPDATE:                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Tuple: (xmin=100, xmax=0) id=1, name='John'                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  After UPDATE by transaction 200:                                    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Dead:  (xmin=100, xmax=200) id=1, name='John'   ← Dead     │    │
│  │  Live:  (xmin=200, xmax=0)   id=1, name='Jane'   ← Current  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  DELETE marks tuple dead (doesn't remove):                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Dead:  (xmin=200, xmax=300) id=1, name='Jane'   ← Dead     │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Problems without VACUUM:                                            │
│  • Dead tuples consume disk space                                    │
│  • Table scans read dead tuples (slower)                            │
│  • Index entries point to dead tuples                               │
│  • Transaction ID wraparound (catastrophic!)                        │
└─────────────────────────────────────────────────────────────────────┘
```

### What VACUUM Does

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VACUUM Operations                                 │
│                                                                      │
│  1. REMOVE DEAD TUPLES                                               │
│     • Scans table for dead tuples                                   │
│     • Marks space as reusable for new tuples                        │
│     • Does NOT return space to OS                                   │
│                                                                      │
│  2. UPDATE FREE SPACE MAP (FSM)                                      │
│     • Tracks available space in each page                           │
│     • Helps INSERT find pages with space                            │
│                                                                      │
│  3. UPDATE VISIBILITY MAP (VM)                                       │
│     • Tracks pages with all-visible tuples                          │
│     • Enables index-only scans                                      │
│                                                                      │
│  4. FREEZE OLD TUPLES                                                │
│     • Sets xmin to FrozenXID for old tuples                         │
│     • Prevents transaction ID wraparound                            │
│                                                                      │
│  5. UPDATE STATISTICS (with ANALYZE)                                 │
│     • Collects column statistics                                     │
│     • Helps query planner make good decisions                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. VACUUM Variants

### Regular VACUUM

```sql
-- Basic VACUUM (non-blocking)
VACUUM tablename;

-- VACUUM with ANALYZE (update statistics)
VACUUM ANALYZE tablename;

-- VACUUM entire database
VACUUM;

-- Verbose output
VACUUM VERBOSE tablename;

-- Behavior:
-- • Concurrent with reads and writes
-- • Marks dead tuple space as reusable
-- • Does NOT shrink table file
-- • Does NOT return space to OS
-- • Lightweight, run frequently
```

### VACUUM FULL

```sql
-- Full vacuum (rewrites table)
VACUUM FULL tablename;

-- Behavior:
-- • EXCLUSIVE lock (blocks all access!)
-- • Rewrites entire table to new file
-- • Returns space to OS
-- • Very slow for large tables
-- • Rebuilds all indexes
-- • Use sparingly (major maintenance only)

-- Space before and after
SELECT pg_size_pretty(pg_relation_size('tablename'));
VACUUM FULL tablename;
SELECT pg_size_pretty(pg_relation_size('tablename'));
```

### VACUUM Options

```sql
-- PostgreSQL 12+ options

-- Skip locked pages (don't wait for conflicting transactions)
VACUUM (SKIP_LOCKED) tablename;

-- Index cleanup options
VACUUM (INDEX_CLEANUP ON) tablename;   -- Default, clean indexes
VACUUM (INDEX_CLEANUP OFF) tablename;  -- Skip index cleanup (faster)
VACUUM (INDEX_CLEANUP AUTO) tablename; -- Decide based on dead tuples

-- Truncate empty pages at end
VACUUM (TRUNCATE ON) tablename;  -- Default
VACUUM (TRUNCATE OFF) tablename; -- Skip truncation

-- Parallel vacuum (PostgreSQL 13+)
VACUUM (PARALLEL 4) tablename;   -- Use 4 workers for index cleanup

-- Combined options
VACUUM (VERBOSE, ANALYZE, PARALLEL 4) tablename;
```

---

## 3. Autovacuum

### How Autovacuum Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Autovacuum Process                                │
│                                                                      │
│  Autovacuum Launcher (1 process)                                     │
│       │                                                              │
│       ├──► Checks tables every autovacuum_naptime (default: 1min)   │
│       │                                                              │
│       └──► Spawns Workers (up to autovacuum_max_workers)            │
│            │                                                         │
│            ▼                                                         │
│  Autovacuum Worker                                                   │
│       │                                                              │
│       ├── Checks thresholds for each table:                         │
│       │   VACUUM if: dead_tuples > threshold + scale * table_size   │
│       │   ANALYZE if: changed_tuples > threshold + scale * table_size│
│       │                                                              │
│       └── Performs VACUUM and/or ANALYZE                            │
│                                                                      │
│  Throttling:                                                         │
│  • autovacuum_vacuum_cost_delay: Sleep between page reads           │
│  • autovacuum_vacuum_cost_limit: Cost budget per round              │
└─────────────────────────────────────────────────────────────────────┘
```

### Autovacuum Configuration

```sql
-- Global settings (postgresql.conf)

-- Enable autovacuum (always keep on!)
autovacuum = on

-- Launcher check interval
autovacuum_naptime = 1min

-- Maximum concurrent workers
autovacuum_max_workers = 3

-- VACUUM thresholds
autovacuum_vacuum_threshold = 50        -- Base dead tuples
autovacuum_vacuum_scale_factor = 0.2    -- + 20% of table size

-- ANALYZE thresholds
autovacuum_analyze_threshold = 50       -- Base changed tuples
autovacuum_analyze_scale_factor = 0.1   -- + 10% of table size

-- Throttling (I/O impact)
autovacuum_vacuum_cost_delay = 2ms      -- Sleep duration
autovacuum_vacuum_cost_limit = 200      -- Cost before sleep

-- Freezing
autovacuum_freeze_max_age = 200000000   -- Force vacuum before this age
```

### Per-Table Settings

```sql
-- Override autovacuum for specific tables

-- High-update table (more aggressive)
ALTER TABLE high_churn SET (
    autovacuum_vacuum_scale_factor = 0.05,    -- 5% instead of 20%
    autovacuum_vacuum_threshold = 100,
    autovacuum_analyze_scale_factor = 0.02,   -- 2% instead of 10%
    autovacuum_vacuum_cost_delay = 0          -- No throttling
);

-- Large table (less aggressive to reduce I/O)
ALTER TABLE huge_table SET (
    autovacuum_vacuum_scale_factor = 0.01,    -- 1% of huge is still big
    autovacuum_vacuum_cost_limit = 100        -- More throttling
);

-- Append-only table (rarely needs vacuum)
ALTER TABLE log_table SET (
    autovacuum_enabled = false  -- Manual vacuum only
);

-- View current settings
SELECT relname, reloptions FROM pg_class WHERE relname = 'high_churn';
```

### Monitoring Autovacuum

```sql
-- Current autovacuum activity
SELECT
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    vacuum_count,
    autovacuum_count,
    n_dead_tup,
    n_live_tup
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Tables needing vacuum
SELECT
    schemaname,
    relname,
    n_dead_tup,
    n_live_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC;

-- Autovacuum progress (PostgreSQL 9.6+)
SELECT * FROM pg_stat_progress_vacuum;
```

---

## 4. Table Bloat

### What is Bloat?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Table Bloat                                       │
│                                                                      │
│  Normal Table:                                                       │
│  ┌────┬────┬────┬────┬────┐                                         │
│  │Live│Live│Live│Live│Live│  100% utilized                          │
│  └────┴────┴────┴────┴────┘                                         │
│                                                                      │
│  Bloated Table:                                                      │
│  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐                │
│  │Live│Free│Live│Free│Free│Live│Free│Free│Live│Free│ 40% utilized  │
│  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘                │
│                                                                      │
│  Causes of bloat:                                                    │
│  • High UPDATE/DELETE rate                                           │
│  • VACUUM not keeping up                                             │
│  • Long-running transactions holding back cleanup                   │
│  • Large UPDATE operations                                          │
│                                                                      │
│  Effects:                                                            │
│  • Wasted disk space                                                 │
│  • Slower sequential scans                                          │
│  • Larger backups                                                    │
│  • Reduced buffer cache efficiency                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Detecting Bloat

```sql
-- Simple dead tuple check
SELECT
    schemaname,
    relname,
    pg_size_pretty(pg_relation_size(relid)) AS size,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- More accurate with pgstattuple extension
CREATE EXTENSION pgstattuple;

SELECT * FROM pgstattuple('tablename');
-- Returns:
-- table_len: Total table size
-- tuple_count: Live tuples
-- tuple_len: Size of live tuples
-- dead_tuple_count: Dead tuples
-- dead_tuple_len: Size of dead tuples
-- free_space: Reusable space
-- free_percent: Percent free space

-- Quick approximation
SELECT * FROM pgstattuple_approx('tablename');
```

### Bloat Estimation Query

```sql
-- Estimate bloat without pgstattuple
WITH constants AS (
    SELECT current_setting('block_size')::numeric AS bs,
           23 AS hdr,  -- Tuple header
           4 AS ma     -- Max align
),
table_stats AS (
    SELECT
        schemaname,
        tablename,
        reltuples::bigint AS row_count,
        relpages::bigint AS page_count
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    JOIN pg_stat_user_tables s ON s.relid = c.oid
    WHERE c.relkind = 'r'
)
SELECT
    schemaname,
    tablename,
    pg_size_pretty((page_count * 8192)::bigint) AS current_size,
    row_count,
    page_count AS current_pages
FROM table_stats
WHERE page_count > 0
ORDER BY page_count DESC
LIMIT 20;
```

---

## 5. Remediation Strategies

### Regular VACUUM

```sql
-- Usually sufficient for normal bloat
VACUUM tablename;
VACUUM ANALYZE tablename;

-- For index bloat
REINDEX INDEX indexname;
REINDEX TABLE tablename;

-- Concurrent reindex (no locks)
REINDEX INDEX CONCURRENTLY indexname;
```

### VACUUM FULL Alternative: pg_repack

```sql
-- pg_repack: Online table rebuild (no locks!)
-- Install extension
CREATE EXTENSION pg_repack;

-- From command line:
-- pg_repack -t tablename dbname

-- Benefits over VACUUM FULL:
-- • No exclusive lock
-- • Table remains accessible during rebuild
-- • Rebuilds indexes too

-- Requirements:
-- • Must have replica identity (primary key or unique)
-- • Needs temporary disk space
```

### CLUSTER

```sql
-- Physically reorder table by index
CLUSTER tablename USING indexname;

-- Exclusive lock like VACUUM FULL
-- But also reorders data for index efficiency

-- Afterward, table ordered by index:
-- Speeds up range scans on that index
```

### Prevention

```sql
-- 1. Tune autovacuum for high-churn tables
ALTER TABLE hot_table SET (
    autovacuum_vacuum_scale_factor = 0.01,
    autovacuum_vacuum_threshold = 100
);

-- 2. Use fillfactor for UPDATE-heavy tables
ALTER TABLE updated_table SET (fillfactor = 80);
-- Leaves 20% free space for HOT updates

-- 3. Partition large tables
-- VACUUM works per partition, more manageable

-- 4. Monitor and alert on dead tuple ratio
-- Alert if dead_pct > 10%
```

---

## 6. Transaction ID Wraparound

### The Danger

```
┌─────────────────────────────────────────────────────────────────────┐
│                    XID Wraparound Emergency                          │
│                                                                      │
│  Transaction IDs are 32-bit, wrap around at ~4 billion              │
│                                                                      │
│  Without freezing old tuples:                                        │
│  • Old tuples suddenly appear "in the future"                       │
│  • Data becomes invisible or corrupted                              │
│                                                                      │
│  Protection:                                                         │
│  • autovacuum_freeze_max_age (default 200M)                         │
│  • When table reaches this age, autovacuum freezes aggressively     │
│  • vacuum_freeze_min_age: Freeze tuples this old                    │
│                                                                      │
│  Emergency:                                                          │
│  • If table reaches 2B XIDs from wrap, database refuses writes      │
│  • Must run VACUUM FREEZE manually                                   │
│  • Single-user mode may be required                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Monitoring XID Age

```sql
-- Database-level XID age
SELECT
    datname,
    age(datfrozenxid) AS xid_age,
    ROUND(100.0 * age(datfrozenxid) / 2147483647, 2) AS pct_toward_wraparound
FROM pg_database
ORDER BY age(datfrozenxid) DESC;

-- Table-level XID age
SELECT
    c.oid::regclass AS table_name,
    age(c.relfrozenxid) AS xid_age,
    pg_size_pretty(pg_relation_size(c.oid)) AS size
FROM pg_class c
WHERE c.relkind = 'r'
ORDER BY age(c.relfrozenxid) DESC
LIMIT 10;

-- Alert threshold: age > 150,000,000 requires attention
```

### Forcing Freeze

```sql
-- Manual freeze
VACUUM FREEZE tablename;

-- Freeze all tables in database
VACUUM FREEZE;

-- Check vacuum_freeze_min_age (when freezing starts)
SHOW vacuum_freeze_min_age;  -- Default: 50000000

-- Set more aggressive freezing for high-write tables
ALTER TABLE high_write SET (
    autovacuum_freeze_max_age = 100000000,
    autovacuum_freeze_min_age = 10000000
);
```

---

## 7. Best Practices

### Autovacuum Configuration

```sql
-- For most workloads, tune these:

-- More workers for large databases
-- postgresql.conf: autovacuum_max_workers = 4

-- Faster response to dead tuples
-- postgresql.conf: autovacuum_naptime = 30s

-- Per-table for hot tables:
ALTER TABLE hot_table SET (
    autovacuum_vacuum_scale_factor = 0.02,
    autovacuum_analyze_scale_factor = 0.01,
    autovacuum_vacuum_cost_delay = 0
);

-- Monitor and adjust based on dead tuple accumulation
```

### Maintenance Windows

```sql
-- Heavy maintenance during low-traffic periods

-- 1. Analyze all tables (update statistics)
ANALYZE;

-- 2. Vacuum with aggressive settings
SET vacuum_cost_delay = 0;
VACUUM ANALYZE;
RESET vacuum_cost_delay;

-- 3. Reindex bloated indexes
REINDEX INDEX CONCURRENTLY idx_bloated;

-- 4. Check and handle bloated tables
-- Use pg_repack for online rebuild if needed
```

### Monitoring Queries

```sql
-- Dead tuple monitor
CREATE VIEW bloat_monitor AS
SELECT
    schemaname,
    relname,
    n_dead_tup,
    n_live_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY dead_pct DESC;

-- XID age monitor
CREATE VIEW xid_monitor AS
SELECT
    c.oid::regclass AS table_name,
    age(c.relfrozenxid) AS xid_age,
    CASE
        WHEN age(c.relfrozenxid) > 150000000 THEN 'CRITICAL'
        WHEN age(c.relfrozenxid) > 100000000 THEN 'WARNING'
        ELSE 'OK'
    END AS status
FROM pg_class c
WHERE c.relkind = 'r'
ORDER BY age(c.relfrozenxid) DESC;
```

---

## Summary

| Operation | Locks | Reclaims Space | When to Use |
|-----------|-------|----------------|-------------|
| VACUUM | None | For reuse only | Frequently (autovacuum) |
| VACUUM FULL | Exclusive | To OS | Severe bloat, downtime OK |
| pg_repack | Minimal | To OS | Severe bloat, no downtime |
| REINDEX | Exclusive/None | Index only | Index bloat |

---

## Further Reading

- PostgreSQL VACUUM documentation
- Autovacuum tuning guide
- pg_repack documentation
