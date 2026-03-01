# CockroachDB

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│              CockroachDB Architecture                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   SQL Layer                          │    │
│  │  Parser → Optimizer → Executor                      │    │
│  │  PostgreSQL wire protocol compatible                │    │
│  └─────────────────────────┬───────────────────────────┘    │
│                            │                                 │
│  ┌─────────────────────────▼───────────────────────────┐    │
│  │              Transaction Layer                       │    │
│  │  MVCC, Distributed transactions, Lock-free reads   │    │
│  └─────────────────────────┬───────────────────────────┘    │
│                            │                                 │
│  ┌─────────────────────────▼───────────────────────────┐    │
│  │              Distribution Layer                      │    │
│  │  Ranges, Raft consensus, Lease holders              │    │
│  └─────────────────────────┬───────────────────────────┘    │
│                            │                                 │
│  ┌─────────────────────────▼───────────────────────────┐    │
│  │              Storage Layer                           │    │
│  │  Pebble (LSM-tree based, RocksDB compatible)        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  All nodes are identical (no special roles)                 │
└─────────────────────────────────────────────────────────────┘
```

## Data Distribution

```
┌─────────────────────────────────────────────────────────────┐
│                 Range-Based Sharding                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Table data split into ranges (default 512MB):             │
│                                                              │
│  ┌───────────────────────────────────────────────────┐      │
│  │ Table: users                                       │      │
│  │                                                    │      │
│  │  Range 1        Range 2        Range 3            │      │
│  │  [A-G]          [H-N]          [O-Z]              │      │
│  │  ┌─────┐        ┌─────┐        ┌─────┐            │      │
│  │  │Alice│        │Helen│        │Oscar│            │      │
│  │  │Bob  │        │Ivan │        │Peter│            │      │
│  │  │Carol│        │John │        │Quinn│            │      │
│  │  │...  │        │...  │        │...  │            │      │
│  │  └─────┘        └─────┘        └─────┘            │      │
│  └───────────────────────────────────────────────────┘      │
│                                                              │
│  Each range replicated 3x (configurable):                  │
│  ┌───────────────────────────────────────────────────┐      │
│  │ Range 1:  Node1(leader) ← Node2 ← Node3          │      │
│  │ Range 2:  Node2(leader) ← Node3 ← Node1          │      │
│  │ Range 3:  Node3(leader) ← Node1 ← Node2          │      │
│  └───────────────────────────────────────────────────┘      │
│                                                              │
│  Automatic splitting when range exceeds threshold          │
│  Automatic rebalancing across nodes                        │
└─────────────────────────────────────────────────────────────┘
```

## Distributed Transactions

```
┌─────────────────────────────────────────────────────────────┐
│           CockroachDB Transaction Model                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  SERIALIZABLE isolation (highest level)                     │
│                                                              │
│  Write Path:                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 1. Gateway node receives transaction               │    │
│  │ 2. Coordinator assigns transaction timestamp       │    │
│  │ 3. Write intents placed on involved ranges         │    │
│  │ 4. Parallel writes with Raft replication          │    │
│  │ 5. Transaction record committed                    │    │
│  │ 6. Intents resolved asynchronously                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Conflict Resolution:                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Write-write conflict:                               │    │
│  │   • Later transaction waits or pushes earlier     │    │
│  │   • Deadlock detection and resolution             │    │
│  │                                                      │    │
│  │ Read-write conflict:                                │    │
│  │   • Reads from MVCC snapshot                       │    │
│  │   • Write skew prevented by read refreshing       │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Raft Consensus

```
┌─────────────────────────────────────────────────────────────┐
│              Raft in CockroachDB                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Each range is a Raft group:                                │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Range 42 Raft Group                                 │    │
│  │                                                      │    │
│  │   ┌──────────────┐                                  │    │
│  │   │   Leader     │ ← Handles all writes            │    │
│  │   │   (Node 1)   │                                  │    │
│  │   └──────┬───────┘                                  │    │
│  │          │ Replicate log entries                   │    │
│  │    ┌─────┴─────┐                                    │    │
│  │    ▼           ▼                                    │    │
│  │ ┌────────┐ ┌────────┐                              │    │
│  │ │Follower│ │Follower│                              │    │
│  │ │(Node 2)│ │(Node 3)│                              │    │
│  │ └────────┘ └────────┘                              │    │
│  │                                                      │    │
│  │ Commit after majority (2/3) acknowledge            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Lease holder: One replica handles reads               │
│  • Usually co-located with Raft leader                     │
│  • Avoids Raft round-trip for reads                        │
└─────────────────────────────────────────────────────────────┘
```

## Multi-Region Deployment

```
┌─────────────────────────────────────────────────────────────┐
│              Multi-Region Capabilities                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  REGIONAL TABLES:                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Data stays in specified region                      │    │
│  │ CREATE TABLE users (...)                            │    │
│  │   LOCALITY REGIONAL BY TABLE IN "us-east";         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  REGIONAL BY ROW:                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Each row assigned to a region                       │    │
│  │ CREATE TABLE users (                                │    │
│  │   id INT,                                           │    │
│  │   region crdb_internal_region,                     │    │
│  │   ...                                               │    │
│  │ ) LOCALITY REGIONAL BY ROW;                        │    │
│  │                                                      │    │
│  │ User in EU → replicas in EU region                 │    │
│  │ User in US → replicas in US region                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  GLOBAL TABLES:                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Replicated everywhere, optimized for reads         │    │
│  │ LOCALITY GLOBAL                                     │    │
│  │ Great for: reference data, configuration           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Key Features Summary

```
┌─────────────────────────────────────────────────────────────┐
│              CockroachDB Key Features                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ✓ PostgreSQL compatible (wire protocol + SQL)             │
│  ✓ Serializable isolation by default                       │
│  ✓ Automatic sharding and rebalancing                      │
│  ✓ Survives node, zone, and region failures               │
│  ✓ Zero-downtime schema changes                            │
│  ✓ Geo-partitioned data for compliance                     │
│  ✓ Change data capture (CDC)                               │
│  ✓ Backup and restore                                      │
│                                                              │
│  Limitations:                                               │
│  • Higher latency than single-node databases               │
│  • Cross-region transactions have higher latency           │
│  • Operational complexity for large clusters               │
└─────────────────────────────────────────────────────────────┘
```
