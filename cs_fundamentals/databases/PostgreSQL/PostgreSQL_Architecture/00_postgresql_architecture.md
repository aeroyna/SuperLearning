# PostgreSQL Architecture

## Overview

PostgreSQL is a powerful, open-source object-relational database system known for its reliability, feature robustness, and standards compliance. Understanding its architecture is essential for optimal configuration, performance tuning, and troubleshooting.

This section covers:

1. **[Process Architecture](01_process_architecture.md)** - Multi-process model and backend processes
2. **[Memory Architecture](02_memory_architecture.md)** - Shared memory and process memory
3. **[Storage and TOAST](03_storage_and_toast.md)** - Data storage, tablespaces, and large objects
4. **[WAL and Checkpoints](04_wal_and_checkpoints.md)** - Write-ahead logging and crash recovery

---

## PostgreSQL Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     PostgreSQL Architecture                              │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                      CLIENT CONNECTIONS                             │ │
│  │         (psql, applications, pgAdmin, etc.)                         │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                    │                                     │
│                                    ▼                                     │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                     POSTMASTER PROCESS                              │ │
│  │              (Main daemon, connection listener)                     │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                    │              │              │                       │
│                    ▼              ▼              ▼                       │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                     │
│  │   Backend    │ │   Backend    │ │   Backend    │  (One per           │
│  │   Process    │ │   Process    │ │   Process    │   connection)       │
│  └──────────────┘ └──────────────┘ └──────────────┘                     │
│           │              │              │                                │
│           └──────────────┼──────────────┘                                │
│                          ▼                                               │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                     SHARED MEMORY                                   │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │ │
│  │  │ Shared       │ │ WAL         │ │ Lock         │ │ Proc       │ │ │
│  │  │ Buffers      │ │ Buffers     │ │ Tables       │ │ Array      │ │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                    │                                     │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                   BACKGROUND PROCESSES                              │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │
│  │  │ WAL      │ │ BG       │ │ Auto-    │ │ Check-   │ │ Stats    │ │ │
│  │  │ Writer   │ │ Writer   │ │ vacuum   │ │ pointer  │ │ Collector│ │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                    │                                     │
│                                    ▼                                     │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                        STORAGE                                      │ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────────┐   │ │
│  │  │ Data Files   │ │ WAL Files    │ │ Configuration Files      │   │ │
│  │  │ (base/)      │ │ (pg_wal/)    │ │ (postgresql.conf, etc.)  │   │ │
│  │  └──────────────┘ └──────────────┘ └──────────────────────────┘   │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Key Differences from MySQL

| Aspect | PostgreSQL | MySQL (InnoDB) |
|--------|------------|----------------|
| Process Model | Multi-process | Multi-threaded |
| Connection | New process per connection | Thread per connection |
| MVCC | Tuple versioning | Undo logs |
| Default Isolation | Read Committed | Repeatable Read |
| Extensibility | Highly extensible | Limited |
| JSON Support | JSONB (binary, indexed) | JSON (text) |

---

## Data Directory Structure

```
$PGDATA/
├── base/                    # Database files
│   ├── 1/                   # template1 database
│   ├── 13395/               # User database (OID)
│   │   ├── 16384            # Table file (relfilenode)
│   │   ├── 16384_fsm        # Free space map
│   │   └── 16384_vm         # Visibility map
├── global/                  # Cluster-wide tables
├── pg_wal/                  # Write-ahead log files
├── pg_xact/                 # Transaction commit status
├── pg_stat/                 # Statistics files
├── pg_stat_tmp/             # Temporary statistics
├── pg_tblspc/               # Tablespace symlinks
├── pg_multixact/            # Multi-transaction status
├── pg_notify/               # LISTEN/NOTIFY data
├── pg_replslot/             # Replication slots
├── pg_snapshots/            # Exported snapshots
├── pg_subtrans/             # Subtransaction status
├── pg_twophase/             # 2PC state files
├── postgresql.conf          # Main configuration
├── pg_hba.conf              # Host-based authentication
├── pg_ident.conf            # Ident authentication mapping
├── postmaster.pid           # PID and lock file
└── postmaster.opts          # Command-line options
```

---

## Quick Reference

```sql
-- Server version
SELECT version();

-- Current database
SELECT current_database();

-- Connection info
SELECT pg_backend_pid(), current_user, inet_server_addr();

-- Configuration
SHOW ALL;
SHOW shared_buffers;

-- Data directory
SHOW data_directory;
```

---

## Version History Highlights

| Version | Key Features |
|---------|-------------|
| PostgreSQL 10 | Logical replication, declarative partitioning |
| PostgreSQL 11 | JIT compilation, stored procedures |
| PostgreSQL 12 | Improved partitioning, generated columns |
| PostgreSQL 13 | Parallel vacuum, incremental sorting |
| PostgreSQL 14 | Connection multiplexing, query pipelining |
| PostgreSQL 15 | MERGE command, JSON logging |
| PostgreSQL 16 | Logical replication improvements |

---

## Learning Path

Start with process architecture to understand PostgreSQL's multi-process model, then explore memory management, storage organization, and finally the crucial WAL system for durability.
