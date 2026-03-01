# MySQL Architecture

## Overview

MySQL is one of the world's most popular open-source relational database management systems. Understanding its architecture is crucial for optimal performance tuning, troubleshooting, and designing scalable database solutions.

This section covers:

1. **[Server Architecture](01_server_architecture.md)** - Client/server model, connection handling, query execution flow
2. **[InnoDB Storage Engine](02_innodb_storage_engine.md)** - The default transactional storage engine
3. **[MyISAM vs InnoDB](03_myisam_vs_innodb.md)** - Storage engine comparison and migration
4. **[MySQL Memory Structures](04_mysql_memory_structures.md)** - Buffer pools, caches, and memory allocation

---

## MySQL Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT CONNECTIONS                           │
│              (MySQL Clients, Applications, Tools)                    │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         CONNECTION LAYER                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │
│  │ Connection Pool │  │ Thread Handling │  │ Authentication      │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────┘  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          SQL LAYER                                   │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌──────────┐  │
│  │  Parser  │→│Preprocessor│→│ Optimizer │→│ Executor │→│  Cache   │  │
│  └──────────┘ └──────────┘ └───────────┘ └──────────┘ └──────────┘  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       STORAGE ENGINE LAYER                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Storage Engine API                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────────┐    │
│  │ InnoDB │  │ MyISAM │  │ Memory │  │ Archive│  │ NDB Cluster│    │
│  └────────┘  └────────┘  └────────┘  └────────┘  └────────────┘    │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        FILE SYSTEM                                   │
│  ┌─────────────┐  ┌────────────┐  ┌────────────┐  ┌─────────────┐   │
│  │ Data Files  │  │ Log Files  │  │ Redo Logs  │  │ Undo Logs   │   │
│  │ (.ibd,.MYD) │  │ (error,    │  │ (ib_log*)  │  │ (undo_*)    │   │
│  │             │  │  slow,     │  │            │  │             │   │
│  │             │  │  general)  │  │            │  │             │   │
│  └─────────────┘  └────────────┘  └────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts

### Pluggable Storage Engine Architecture

MySQL's unique feature is its pluggable storage engine architecture. The SQL layer handles:
- Query parsing and optimization
- Caching
- Built-in functions
- Stored procedures

Storage engines handle:
- Data storage and retrieval
- Index management
- Transaction support (engine-dependent)
- Locking mechanisms

### Default Storage Engine: InnoDB

Since MySQL 5.5, InnoDB is the default storage engine, offering:
- Full ACID compliance
- Row-level locking
- Foreign key support
- MVCC (Multi-Version Concurrency Control)
- Crash recovery

---

## Version History Highlights

| Version | Key Features |
|---------|-------------|
| MySQL 5.5 | InnoDB default, semi-sync replication |
| MySQL 5.6 | Full-text search in InnoDB, GTID replication |
| MySQL 5.7 | Native JSON support, sys schema, group replication |
| MySQL 8.0 | Data dictionary, roles, CTEs, window functions, invisible indexes |

---

## Learning Path

Start with server architecture fundamentals, then deep dive into InnoDB internals, compare storage engines, and finally understand memory management for optimal configuration.
