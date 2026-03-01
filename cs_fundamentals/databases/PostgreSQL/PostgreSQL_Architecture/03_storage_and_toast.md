# PostgreSQL Storage and TOAST

## Learning Objectives
- Understand PostgreSQL's storage hierarchy
- Master tablespaces and file organization
- Learn TOAST (The Oversized-Attribute Storage Technique)
- Manage storage effectively

---

## 1. Storage Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Storage Hierarchy                          │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                        CLUSTER                                      │ │
│  │              (One PostgreSQL instance)                              │ │
│  │                                                                     │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │                     DATABASES                                 │  │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐                       │  │ │
│  │  │  │template0│  │template1│  │  mydb   │                       │  │ │
│  │  │  └─────────┘  └─────────┘  └────┬────┘                       │  │ │
│  │  └─────────────────────────────────┼────────────────────────────┘  │ │
│  │                                    │                                │ │
│  │                                    ▼                                │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │                      SCHEMAS                                  │  │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐                       │  │ │
│  │  │  │ public  │  │ schema1 │  │ schema2 │                       │  │ │
│  │  │  └────┬────┘  └─────────┘  └─────────┘                       │  │ │
│  │  └───────┼──────────────────────────────────────────────────────┘  │ │
│  │          │                                                          │ │
│  │          ▼                                                          │ │
│  │  ┌──────────────────────────────────────────────────────────────┐  │ │
│  │  │                     RELATIONS                                 │  │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │  │ │
│  │  │  │ Tables  │  │ Indexes │  │ Views   │  │ Materialized    │  │  │ │
│  │  │  │         │  │         │  │         │  │ Views           │  │  │ │
│  │  │  └─────────┘  └─────────┘  └─────────┘  └─────────────────┘  │  │ │
│  │  └──────────────────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. File Organization

### Data Directory Structure

```bash
$PGDATA/
├── base/                          # Database directories
│   ├── 1/                         # template1 (OID 1)
│   ├── 13395/                     # postgres database
│   └── 16384/                     # User database (OID)
│       ├── 16385                  # Table file (relfilenode)
│       ├── 16385.1                # First 1GB segment
│       ├── 16385_fsm              # Free space map
│       ├── 16385_vm               # Visibility map
│       └── 16386                  # Another relation
├── global/                        # Cluster-wide tables
│   ├── pg_control                 # Control file
│   ├── pg_filenode.map            # OID to file mapping
│   └── 1213                       # pg_database table
├── pg_wal/                        # WAL files
│   ├── 000000010000000000000001   # WAL segment
│   └── archive_status/            # Archive status
├── pg_tblspc/                     # Tablespace symlinks
└── pg_stat_tmp/                   # Temp statistics files
```

### Finding File Locations

```sql
-- Database OID
SELECT oid, datname FROM pg_database;

-- Table file location
SELECT
    pg_relation_filepath('tablename') AS filepath,
    relfilenode,
    reltablespace
FROM pg_class
WHERE relname = 'tablename';

-- All relation files
SELECT
    c.relname,
    c.relkind,
    c.relfilenode,
    pg_relation_filepath(c.oid) AS filepath,
    pg_size_pretty(pg_relation_size(c.oid)) AS size
FROM pg_class c
WHERE c.relnamespace = 'public'::regnamespace
ORDER BY pg_relation_size(c.oid) DESC
LIMIT 20;
```

---

## 3. Page Structure

PostgreSQL uses 8KB pages (blocks) as the basic unit of storage.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Page Structure (8KB = 8192 bytes)                     │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ Page Header (24 bytes)                                              │ │
│  │ ┌────────────────────────────────────────────────────────────────┐ │ │
│  │ │ pd_lsn (8) │ pd_checksum (2) │ pd_flags (2) │ pd_lower (2) │   │ │ │
│  │ │ pd_upper (2) │ pd_special (2) │ pd_pagesize_version (2) │ ...  │ │ │
│  │ └────────────────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ Line Pointers (Item IDs)                                            │ │
│  │ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐                            │ │
│  │ │ lp 1  │ │ lp 2  │ │ lp 3  │ │ lp 4  │  → Points to tuples       │ │
│  │ │offset │ │offset │ │offset │ │offset │                            │ │
│  │ │length │ │length │ │length │ │length │                            │ │
│  │ └───────┘ └───────┘ └───────┘ └───────┘                            │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                             │                                            │
│                             │ pd_lower (free space starts)               │
│                             ▼                                            │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                         FREE SPACE                                  │ │
│  │                    (grows as tuples added)                          │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                             ▲                                            │
│                             │ pd_upper (free space ends)                 │
│                             │                                            │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ Tuples (Row Data)                                                   │ │
│  │ ┌──────────────────────────────────────────────────────────────┐   │ │
│  │ │ Tuple 4 │ Tuple 3 │ Tuple 2 │ Tuple 1 │                      │   │ │
│  │ │ (newest)│         │         │ (oldest)│  ← Grows backwards   │   │ │
│  │ └──────────────────────────────────────────────────────────────┘   │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ Special Space (for indexes, 0 for tables)                           │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### Tuple Header

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    Tuple Header (23+ bytes)                              │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ t_xmin (4 bytes)     - Transaction ID that inserted this tuple     │ │
│  │ t_xmax (4 bytes)     - Transaction ID that deleted/updated         │ │
│  │ t_cid (4 bytes)      - Command ID within transaction               │ │
│  │ t_ctid (6 bytes)     - Current tuple ID (for updates: new version) │ │
│  │ t_infomask (2 bytes) - Tuple flags                                 │ │
│  │ t_infomask2 (2 bytes) - More flags + number of attributes          │ │
│  │ t_hoff (1 byte)      - Offset to user data                         │ │
│  │ [null bitmap]        - Optional, if any NULLs                      │ │
│  │ [OID]                - Optional, if table has OIDs                 │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  Total header: 23 bytes minimum + null bitmap + padding                  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 4. TOAST (The Oversized-Attribute Storage Technique)

### Why TOAST?

- Maximum row size: ~8KB (page size minus overhead)
- Large columns need special handling
- TOAST compresses and/or stores externally

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    TOAST Mechanism                                       │
│                                                                          │
│  Main Table                                                              │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ id │ name │ small_data │ large_text_column                      │    │
│  │ 1  │ foo  │ abc        │ [TOAST pointer → pg_toast_12345]       │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                     │                                    │
│                                     ▼                                    │
│  TOAST Table (pg_toast.pg_toast_12345)                                   │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │ chunk_id │ chunk_seq │ chunk_data                                │    │
│  │    1     │     0     │ [compressed data chunk 1]                 │    │
│  │    1     │     1     │ [compressed data chunk 2]                 │    │
│  │    1     │     2     │ [compressed data chunk 3]                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  TOAST threshold: ~2KB (TOAST_TUPLE_THRESHOLD)                           │
│  Chunk size: ~2000 bytes (TOAST_MAX_CHUNK_SIZE)                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### TOAST Strategies

```sql
-- View column storage types
SELECT
    attname,
    atttypid::regtype AS type,
    CASE attstorage
        WHEN 'p' THEN 'plain (no TOAST)'
        WHEN 'e' THEN 'external (uncompressed)'
        WHEN 'x' THEN 'extended (compressed, then external)'
        WHEN 'm' THEN 'main (compressed, inline preferred)'
    END AS storage
FROM pg_attribute
WHERE attrelid = 'mytable'::regclass
AND attnum > 0;

-- Change storage strategy
ALTER TABLE mytable ALTER COLUMN large_text SET STORAGE EXTERNAL;
ALTER TABLE mytable ALTER COLUMN large_text SET STORAGE EXTENDED;
```

### TOAST Strategies Explained

| Strategy | Compression | External Storage | Use Case |
|----------|-------------|-----------------|----------|
| PLAIN | No | No | Fixed-size types (int, etc.) |
| EXTERNAL | No | Yes | Already compressed data |
| EXTENDED | Yes | Yes | Default for variable-length |
| MAIN | Yes | Last resort | Keep inline if possible |

### Viewing TOAST Tables

```sql
-- Find TOAST table for a relation
SELECT
    c.relname AS table_name,
    t.relname AS toast_table,
    pg_size_pretty(pg_relation_size(t.oid)) AS toast_size
FROM pg_class c
JOIN pg_class t ON c.reltoastrelid = t.oid
WHERE c.relname = 'mytable';

-- TOAST table statistics
SELECT
    c.relname,
    pg_size_pretty(pg_relation_size(c.oid)) AS main_size,
    pg_size_pretty(pg_relation_size(c.reltoastrelid)) AS toast_size,
    pg_size_pretty(pg_total_relation_size(c.oid)) AS total_size
FROM pg_class c
WHERE c.relkind = 'r' AND c.reltoastrelid != 0
ORDER BY pg_total_relation_size(c.oid) DESC
LIMIT 10;
```

---

## 5. Tablespaces

### Creating and Using Tablespaces

```sql
-- Create tablespace
CREATE TABLESPACE fast_storage LOCATION '/mnt/ssd/pgdata';
CREATE TABLESPACE archive_storage LOCATION '/mnt/hdd/pgdata';

-- Create table in specific tablespace
CREATE TABLE hot_data (id int, data text) TABLESPACE fast_storage;

-- Create index in tablespace
CREATE INDEX idx_data ON hot_data(data) TABLESPACE fast_storage;

-- Move table to tablespace
ALTER TABLE mytable SET TABLESPACE fast_storage;

-- Set default tablespace for database
ALTER DATABASE mydb SET TABLESPACE fast_storage;

-- View tablespaces
SELECT * FROM pg_tablespace;

-- Tablespace size
SELECT
    spcname,
    pg_size_pretty(pg_tablespace_size(oid)) AS size
FROM pg_tablespace;
```

### Tablespace Use Cases

```sql
-- Separate tablespaces for:
-- 1. Hot vs cold data
CREATE TABLESPACE hot_ts LOCATION '/mnt/nvme/pg';
CREATE TABLESPACE cold_ts LOCATION '/mnt/hdd/pg';

-- 2. Indexes on fast storage
CREATE TABLESPACE idx_ts LOCATION '/mnt/ssd/pg_idx';
CREATE INDEX idx_important ON data(col) TABLESPACE idx_ts;

-- 3. Temporary files
-- postgresql.conf: temp_tablespaces = 'temp_ts'
CREATE TABLESPACE temp_ts LOCATION '/mnt/fast/temp';
```

---

## 6. Free Space Map (FSM)

```sql
-- Each table has an FSM file: tablefile_fsm
-- Tracks free space in each page

-- View free space
CREATE EXTENSION pg_freespacemap;

SELECT
    blkno,
    avail,
    ROUND(100 * avail / 8192.0, 2) AS pct_free
FROM pg_freespace('mytable')
ORDER BY blkno
LIMIT 20;

-- Total free space in table
SELECT
    pg_size_pretty(SUM(avail)) AS free_space,
    pg_size_pretty(pg_relation_size('mytable')) AS total_size,
    ROUND(100.0 * SUM(avail) / pg_relation_size('mytable'), 2) AS pct_free
FROM pg_freespace('mytable');
```

---

## 7. Visibility Map (VM)

```sql
-- Each table has a VM file: tablefile_vm
-- Tracks which pages have only visible tuples

-- Benefits:
-- 1. Index-only scans can skip heap fetch
-- 2. VACUUM can skip all-visible pages

-- Check visibility map coverage
SELECT
    relname,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 10;
```

---

## 8. Storage Monitoring

### Table Sizes

```sql
-- Table and index sizes
SELECT
    schemaname,
    relname,
    pg_size_pretty(pg_relation_size(relid)) AS table_size,
    pg_size_pretty(pg_indexes_size(relid)) AS indexes_size,
    pg_size_pretty(pg_total_relation_size(relid)) AS total_size
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC
LIMIT 20;

-- Database size
SELECT
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Bloat estimation
SELECT
    schemaname, tablename,
    ROUND((CASE WHEN otta=0 THEN 0.0 ELSE sml.relpages/otta::numeric END),1) AS tbloat,
    pg_size_pretty(CASE WHEN relpages < otta THEN 0
        ELSE (bs*(sml.relpages-otta))::bigint END) AS wastedbytes
FROM (
    -- Complex bloat estimation query...
) AS sml;
```

### Disk Usage Alerts

```sql
-- Create function to check disk usage
CREATE OR REPLACE FUNCTION check_disk_usage()
RETURNS TABLE (tablespace_name text, total_size text, pct_used numeric) AS $$
BEGIN
    RETURN QUERY
    SELECT
        spcname::text,
        pg_size_pretty(pg_tablespace_size(oid)),
        -- Note: Actual disk usage requires OS-level checks
        0.0::numeric
    FROM pg_tablespace;
END;
$$ LANGUAGE plpgsql;
```

---

## Summary

| Component | Purpose | File Extension |
|-----------|---------|----------------|
| Main file | Table/index data | (none) |
| FSM | Free space tracking | _fsm |
| VM | Visibility tracking | _vm |
| TOAST | Large values | Separate table |
| Segment | 1GB file splits | .1, .2, etc. |

---

## Further Reading

- PostgreSQL Storage documentation
- "PostgreSQL Internals" book
- TOAST technical documentation
