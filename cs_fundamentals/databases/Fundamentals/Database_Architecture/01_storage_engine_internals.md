# Storage Engine Internals

## 1. Introduction

A **storage engine** is the component of a database that handles how data is stored, retrieved, and managed on disk. Different storage engines offer different trade-offs between performance, reliability, and features.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    STORAGE ENGINE OVERVIEW                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                      SQL Layer                                   │      │
│   │           (Parser, Optimizer, Executor)                         │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                              │                                              │
│                              ▼                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                    Storage Engine API                            │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│          │              │              │              │                     │
│          ▼              ▼              ▼              ▼                     │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐               │
│   │  InnoDB  │   │  MyISAM  │   │  RocksDB │   │   Heap   │               │
│   └──────────┘   └──────────┘   └──────────┘   └──────────┘               │
│                                                                              │
│   Storage engines are pluggable in MySQL                                    │
│   PostgreSQL has a single integrated storage engine                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Storage Structures

### 2.1 Pages and Blocks

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PAGE STRUCTURE                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Databases read/write data in fixed-size units called PAGES               │
│                                                                              │
│   Common page sizes:                                                        │
│   • PostgreSQL: 8 KB (default)                                             │
│   • MySQL/InnoDB: 16 KB (default)                                          │
│   • SQL Server: 8 KB                                                       │
│   • Oracle: 8 KB (configurable)                                            │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────┐              │
│   │                    PAGE (16 KB)                          │              │
│   ├─────────────────────────────────────────────────────────┤              │
│   │ Page Header (metadata, checksums, LSN)           ~100B  │              │
│   ├─────────────────────────────────────────────────────────┤              │
│   │ Row Directory (pointers to rows)                        │              │
│   ├─────────────────────────────────────────────────────────┤              │
│   │                                                          │              │
│   │                   Row Data                               │              │
│   │            (actual table rows)                           │              │
│   │                                                          │              │
│   ├─────────────────────────────────────────────────────────┤              │
│   │ Free Space                                              │              │
│   ├─────────────────────────────────────────────────────────┤              │
│   │ Page Trailer (checksums)                                │              │
│   └─────────────────────────────────────────────────────────┘              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Row Storage Formats

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ROW STORAGE FORMATS                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   N-ARY STORAGE MODEL (NSM) - Row-Oriented                                 │
│   ─────────────────────────────────────────                                 │
│   All columns of a row stored together                                     │
│                                                                              │
│   Page: [Row1: id,name,age,salary][Row2: id,name,age,salary][...]          │
│                                                                              │
│   ✓ Good for OLTP (read/write entire rows)                                │
│   ✓ Efficient for point queries                                           │
│   ✗ Reads unnecessary columns for analytics                               │
│                                                                              │
│   Used by: PostgreSQL, MySQL, Oracle, SQL Server                           │
│                                                                              │
│   ─────────────────────────────────────────────────────────────────────    │
│                                                                              │
│   DECOMPOSITION STORAGE MODEL (DSM) - Column-Oriented                      │
│   ───────────────────────────────────────────────────                      │
│   Each column stored separately                                             │
│                                                                              │
│   File1: [id1, id2, id3, id4, ...]                                         │
│   File2: [name1, name2, name3, name4, ...]                                 │
│   File3: [age1, age2, age3, age4, ...]                                     │
│                                                                              │
│   ✓ Excellent for analytics (read only needed columns)                    │
│   ✓ Better compression (similar values together)                          │
│   ✗ Expensive for row-level operations                                    │
│                                                                              │
│   Used by: ClickHouse, Redshift, BigQuery, Parquet files                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. B-Tree Storage

### 3.1 B-Tree Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        B-TREE STRUCTURE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   B-Trees are the primary index structure for most databases               │
│                                                                              │
│                         ┌─────────────────┐                                 │
│                         │   Root Page     │                                 │
│                         │  [30 | 60 | 90] │                                 │
│                         └────────┬────────┘                                 │
│                    ┌─────────────┼─────────────┐                            │
│                    ▼             ▼             ▼                            │
│           ┌────────────┐ ┌────────────┐ ┌────────────┐                     │
│           │ [10|15|20] │ │ [40|45|50] │ │ [70|75|80] │                     │
│           └─────┬──────┘ └─────┬──────┘ └─────┬──────┘                     │
│                 │              │              │                             │
│                 ▼              ▼              ▼                             │
│           ┌──────────┐   ┌──────────┐   ┌──────────┐                       │
│           │Leaf Pages│   │Leaf Pages│   │Leaf Pages│                       │
│           │(actual   │   │(actual   │   │(actual   │                       │
│           │ data)    │   │ data)    │   │ data)    │                       │
│           └──────────┘   └──────────┘   └──────────┘                       │
│                                                                              │
│   Properties:                                                               │
│   • Balanced: All leaf nodes at same depth                                 │
│   • Sorted: Keys in order for range scans                                  │
│   • Self-balancing: Splits/merges maintain balance                        │
│   • Height: log(N) for N keys                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Clustered vs Non-Clustered

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              CLUSTERED vs NON-CLUSTERED INDEX                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   CLUSTERED INDEX (Primary Key Index)                                       │
│   ────────────────────────────────────                                      │
│   • Leaf nodes contain actual row data                                     │
│   • Table data is physically ordered by this key                           │
│   • Only ONE clustered index per table                                     │
│   • Called "Index-Organized Table" in Oracle                               │
│                                                                              │
│   ┌─────────────────┐                                                       │
│   │    B-Tree       │                                                       │
│   │   (index)       │                                                       │
│   └────────┬────────┘                                                       │
│            ▼                                                                 │
│   ┌─────────────────────────────────────────────────────┐                   │
│   │ Leaf: [key1 → row_data1][key2 → row_data2][...]     │                   │
│   └─────────────────────────────────────────────────────┘                   │
│                                                                              │
│   NON-CLUSTERED INDEX (Secondary Index)                                     │
│   ─────────────────────────────────────                                     │
│   • Leaf nodes contain pointers to actual rows                             │
│   • Multiple secondary indexes allowed                                     │
│   • Requires extra lookup to get row data                                  │
│                                                                              │
│   ┌─────────────────┐      ┌─────────────────┐                             │
│   │ Secondary Index │      │   Table Data    │                             │
│   │ [key → PK_ptr]  │─────►│ (or Heap)       │                             │
│   └─────────────────┘      └─────────────────┘                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Heap vs Index-Organized Tables

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   HEAP vs INDEX-ORGANIZED                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   HEAP TABLE                                                                │
│   ──────────                                                                │
│   • Rows stored in no particular order                                     │
│   • New rows added wherever space exists                                   │
│   • Indexes point directly to row location (TID/ROWID)                    │
│   • Used by: PostgreSQL, Oracle (default)                                  │
│                                                                              │
│   ┌────────────────────────────────────────┐                               │
│   │ Heap: [row3][row1][empty][row5][row2]  │                               │
│   └────────────────────────────────────────┘                               │
│         ▲                                                                   │
│         │ TID (page, offset)                                               │
│   ┌─────┴──────┐                                                           │
│   │   Index    │                                                           │
│   └────────────┘                                                           │
│                                                                              │
│   INDEX-ORGANIZED TABLE (IOT)                                               │
│   ───────────────────────────                                               │
│   • Rows stored sorted by primary key                                      │
│   • No separate heap needed                                                │
│   • Secondary indexes point to PK value                                    │
│   • Used by: MySQL/InnoDB, Oracle (option)                                 │
│                                                                              │
│   ┌────────────────────────────────────────┐                               │
│   │ IOT: [row1][row2][row3][row4][row5]    │  ← Sorted by PK              │
│   └────────────────────────────────────────┘                               │
│         ▲                                                                   │
│         │ PK value                                                         │
│   ┌─────┴──────┐                                                           │
│   │ Sec Index  │                                                           │
│   └────────────┘                                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Log-Structured Storage (LSM Trees)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      LSM TREE STRUCTURE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Log-Structured Merge Tree - optimized for write-heavy workloads          │
│                                                                              │
│   WRITE PATH:                                                               │
│   ┌──────────────────────────────────────────────────────────────────┐     │
│   │                                                                   │     │
│   │   1. Write to MemTable (in-memory, sorted)                       │     │
│   │   2. When MemTable full → Flush to SSTable on disk               │     │
│   │   3. Background compaction merges SSTables                       │     │
│   │                                                                   │     │
│   └──────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│   ┌───────────────┐                                                         │
│   │   MemTable    │  (RAM - Red-Black Tree or Skip List)                   │
│   │ [k3:v][k5:v]  │                                                         │
│   └───────┬───────┘                                                         │
│           │ Flush                                                           │
│           ▼                                                                 │
│   ┌───────────────┐  Level 0 (newest)                                      │
│   │  SSTable-0    │  ← Immutable, sorted                                   │
│   └───────────────┘                                                         │
│   ┌───────────────┐  Level 1                                               │
│   │  SSTable-1    │                                                         │
│   └───────────────┘                                                         │
│   ┌───────────────────────────┐  Level 2 (oldest, largest)                 │
│   │       SSTable-2           │                                             │
│   └───────────────────────────┘                                             │
│                                                                              │
│   READ PATH:                                                                │
│   1. Check MemTable                                                        │
│   2. Check SSTables from newest to oldest                                  │
│   3. Use Bloom filters to skip SSTables                                    │
│                                                                              │
│   Used by: RocksDB, LevelDB, Cassandra, HBase                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Data File Organization

### 6.1 PostgreSQL

```
Data Directory Structure:
$PGDATA/
├── base/                    # Database files
│   ├── 1/                   # Database OID 1 (template1)
│   ├── 13067/               # Database OID 13067
│   │   ├── 16384            # Table file (relfilenode)
│   │   ├── 16384_fsm        # Free space map
│   │   ├── 16384_vm         # Visibility map
│   │   └── 16387            # Index file
│   └── ...
├── global/                  # Cluster-wide tables
├── pg_wal/                  # Write-ahead log
├── pg_xact/                 # Transaction status
└── pg_stat/                 # Statistics
```

### 6.2 MySQL/InnoDB

```
Data Directory Structure:
/var/lib/mysql/
├── ibdata1                  # System tablespace
├── ib_logfile0              # Redo log
├── ib_logfile1              # Redo log
├── undo_001                 # Undo tablespace
├── mydb/                    # Database directory
│   ├── users.ibd            # Table data + indexes
│   ├── orders.ibd           # Table data + indexes
│   └── ...
└── mysql/                   # System database
```

---

## 7. Compression

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DATA COMPRESSION                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PAGE COMPRESSION:                                                         │
│   • Compress entire pages                                                  │
│   • Transparent to queries                                                 │
│   • Trade-off: CPU vs I/O                                                  │
│                                                                              │
│   -- PostgreSQL                                                             │
│   CREATE TABLE logs (...) WITH (toast_tuple_target = 128);                 │
│   -- Large values compressed with TOAST                                    │
│                                                                              │
│   -- MySQL/InnoDB                                                           │
│   CREATE TABLE logs (...) ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;         │
│                                                                              │
│   COLUMN COMPRESSION:                                                       │
│   • Compress individual columns                                            │
│   • Good for repetitive data                                               │
│                                                                              │
│   -- PostgreSQL 14+                                                         │
│   ALTER TABLE logs ALTER COLUMN json_data SET COMPRESSION lz4;             │
│                                                                              │
│   Compression methods:                                                      │
│   • LZ4: Fast, moderate compression                                       │
│   • ZSTD: Good balance of speed and ratio                                 │
│   • pglz: PostgreSQL default (slower but no dependency)                   │
│   • Snappy: Very fast, used by RocksDB                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Summary

| Storage Model | Characteristics | Best For |
|---------------|-----------------|----------|
| Row-Oriented (NSM) | Rows stored together | OLTP, point queries |
| Column-Oriented (DSM) | Columns stored separately | Analytics, aggregations |
| Heap | Unordered, fast inserts | General purpose |
| Index-Organized | Ordered by PK | Range scans on PK |
| LSM Tree | Write-optimized | Write-heavy workloads |

**Key Concepts:**
- Pages are the fundamental I/O unit
- B-Trees provide logarithmic access time
- Clustered indexes store data in sorted order
- LSM trees optimize for writes by batching
- Compression trades CPU for storage/I/O
