# TiDB

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    TiDB Architecture                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   TiDB Servers                       │    │
│  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐            │    │
│  │  │ TiDB │  │ TiDB │  │ TiDB │  │ TiDB │            │    │
│  │  └──────┘  └──────┘  └──────┘  └──────┘            │    │
│  │  Stateless SQL layer, MySQL compatible              │    │
│  └──────────────────────┬──────────────────────────────┘    │
│                         │                                    │
│  ┌──────────────────────▼──────────────────────────────┐    │
│  │                 Placement Driver (PD)                │    │
│  │  ┌────┐  ┌────┐  ┌────┐                             │    │
│  │  │ PD │  │ PD │  │ PD │   (Raft group)             │    │
│  │  └────┘  └────┘  └────┘                             │    │
│  │  Metadata, timestamp oracle, scheduling             │    │
│  └──────────────────────┬──────────────────────────────┘    │
│                         │                                    │
│  ┌──────────────────────▼──────────────────────────────┐    │
│  │                   TiKV Cluster                       │    │
│  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐            │    │
│  │  │ TiKV │  │ TiKV │  │ TiKV │  │ TiKV │            │    │
│  │  └──────┘  └──────┘  └──────┘  └──────┘            │    │
│  │  Distributed key-value storage (Raft + RocksDB)     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Optional: TiFlash (columnar analytics)                     │
└─────────────────────────────────────────────────────────────┘
```

## Component Details

```
┌─────────────────────────────────────────────────────────────┐
│                    TiDB Components                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TiDB Server:                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Stateless, can scale horizontally                 │    │
│  │ • MySQL protocol compatible                         │    │
│  │ • SQL parsing, optimization, execution              │    │
│  │ • Converts SQL to KV operations                     │    │
│  │ • No persistent state                               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Placement Driver (PD):                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Cluster metadata storage                          │    │
│  │ • Timestamp allocation (TSO)                        │    │
│  │ • Data placement and scheduling                     │    │
│  │ • Load balancing                                    │    │
│  │ • Runs as Raft group (3 or 5 nodes)                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TiKV:                                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Distributed key-value store                       │    │
│  │ • Data split into regions (~96MB each)             │    │
│  │ • Each region replicated via Raft                  │    │
│  │ • MVCC for concurrency control                      │    │
│  │ • RocksDB as storage engine                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TiFlash (Optional):                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Columnar replica of TiKV data                     │    │
│  │ • Optimized for OLAP queries                        │    │
│  │ • Real-time replication from TiKV                   │    │
│  │ • HTAP: same data for OLTP and OLAP               │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Transaction Model (Percolator)

```
┌─────────────────────────────────────────────────────────────┐
│              Percolator Transaction Model                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Based on Google's Percolator paper                         │
│  Optimistic concurrency control with MVCC                   │
│                                                              │
│  Transaction Flow:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 1. BEGIN                                             │    │
│  │    • Get start timestamp (start_ts) from PD        │    │
│  │                                                      │    │
│  │ 2. READ                                              │    │
│  │    • Read data at start_ts                          │    │
│  │    • Sees consistent snapshot                       │    │
│  │                                                      │    │
│  │ 3. PREWRITE (first phase)                           │    │
│  │    • Write tentative values with locks             │    │
│  │    • Check for conflicts                            │    │
│  │    • One key designated as "primary"               │    │
│  │                                                      │    │
│  │ 4. COMMIT (second phase)                            │    │
│  │    • Get commit timestamp (commit_ts) from PD      │    │
│  │    • Write commit record to primary                 │    │
│  │    • Asynchronously clean up secondaries           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Lock Types:                                                │
│  • Lock: Tentative write, points to primary               │
│  • Write: Commit record with commit_ts                     │
│  • Data: Actual row data versioned by start_ts            │
└─────────────────────────────────────────────────────────────┘
```

## HTAP Architecture

```
┌─────────────────────────────────────────────────────────────┐
│         Hybrid Transactional/Analytical Processing           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                     TiDB Server                      │    │
│  │  ┌──────────────────┬──────────────────────────┐    │    │
│  │  │   OLTP Queries   │      OLAP Queries        │    │    │
│  │  │   (Point lookups,│      (Aggregations,      │    │    │
│  │  │    transactions) │       analytics)         │    │    │
│  │  └────────┬─────────┴───────────┬──────────────┘    │    │
│  └───────────┼─────────────────────┼────────────────────┘    │
│              │                     │                         │
│              ▼                     ▼                         │
│  ┌───────────────────┐  ┌─────────────────────────┐         │
│  │       TiKV        │  │        TiFlash          │         │
│  │   (Row storage)   │  │   (Column storage)      │         │
│  │                   │  │                         │         │
│  │   ┌─────────────┐ │  │   ┌─────────────────┐   │         │
│  │   │   Region    │ │  │   │    Columnar     │   │         │
│  │   │  (B-tree)   │─┼──┼──▶│    Replica      │   │         │
│  │   └─────────────┘ │  │   └─────────────────┘   │         │
│  │   Row-oriented    │  │   Column-oriented       │         │
│  └───────────────────┘  └─────────────────────────┘         │
│                                                              │
│  Raft Learner: TiFlash receives updates as Raft learner    │
│  Optimizer: Automatically chooses TiKV or TiFlash          │
└─────────────────────────────────────────────────────────────┘
```

## MySQL Compatibility

```
┌─────────────────────────────────────────────────────────────┐
│              MySQL Compatibility                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  COMPATIBLE:                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✓ MySQL wire protocol                               │    │
│  │ ✓ Most SQL syntax                                   │    │
│  │ ✓ Indexes (B-tree, unique)                          │    │
│  │ ✓ Transactions (Repeatable Read, Read Committed)   │    │
│  │ ✓ Common functions and operators                    │    │
│  │ ✓ Views, stored procedures (basic)                  │    │
│  │ ✓ MySQL client tools and drivers                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  DIFFERENCES:                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • AUTO_INCREMENT gaps (not sequential)              │    │
│  │ • Some functions behave differently                 │    │
│  │ • Foreign keys (experimental)                       │    │
│  │ • Triggers (limited support)                        │    │
│  │ • Storage engines (TiKV only, no InnoDB/MyISAM)    │    │
│  │ • Character sets (limited compared to MySQL)       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Migration: Use TiDB Data Migration (DM) for MySQL → TiDB  │
└─────────────────────────────────────────────────────────────┘
```

## Key Features Summary

```
┌─────────────────────────────────────────────────────────────┐
│                  TiDB Key Features                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ✓ MySQL compatible (protocol and syntax)                  │
│  ✓ Horizontal scaling (stateless SQL layer)                │
│  ✓ HTAP with TiFlash columnar engine                       │
│  ✓ Strong consistency (Raft replication)                   │
│  ✓ Online DDL without blocking                             │
│  ✓ Automatic failover                                      │
│  ✓ TiDB Cloud (managed service)                            │
│  ✓ Tools: DM (migration), BR (backup), TiCDC (CDC)        │
│                                                              │
│  Best for:                                                  │
│  • MySQL workloads that need to scale out                  │
│  • Combined OLTP and OLAP workloads                        │
│  • Real-time analytics on transactional data               │
└─────────────────────────────────────────────────────────────┘
```
