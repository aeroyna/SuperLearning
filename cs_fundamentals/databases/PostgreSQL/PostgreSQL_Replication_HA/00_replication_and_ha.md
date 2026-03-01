# PostgreSQL Replication and High Availability

## Overview

PostgreSQL provides robust replication and high availability features essential for production deployments. This section covers streaming replication, logical replication, HA patterns, and backup strategies.

---

## What You'll Learn

### 1. Streaming Replication
- Physical replication fundamentals
- Primary-standby setup
- Synchronous vs asynchronous replication
- Replication slots and WAL retention

### 2. Logical Replication
- Publish-subscribe model
- Selective table replication
- Cross-version replication
- Use cases and limitations

### 3. High Availability Patterns
- Failover strategies
- Patroni and pg_auto_failover
- Connection pooling with PgBouncer
- Multi-region deployments

### 4. Backup and Recovery
- pg_dump and pg_dumpall
- pg_basebackup
- Point-in-time recovery (PITR)
- Backup tools (pgBackRest, Barman)

---

## Replication Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Replication Options                    │
│                                                                      │
│  STREAMING REPLICATION (Physical)                                    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Byte-for-byte copy of entire cluster                     │    │
│  │  • WAL records streamed to standby                          │    │
│  │  • Read-only queries on standby (hot standby)               │    │
│  │  • Standby can become new primary (failover)                │    │
│  │  • Same PostgreSQL version required                         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  LOGICAL REPLICATION                                                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Replicates logical changes (INSERT, UPDATE, DELETE)      │    │
│  │  • Selective: Choose tables/columns to replicate            │    │
│  │  • Cross-version compatible (within limits)                 │    │
│  │  • Subscriber can have additional tables/indexes            │    │
│  │  • Bi-directional replication possible                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Comparison:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Feature          │ Streaming        │ Logical             │    │
│  │  ─────────────────────────────────────────────────────────  │    │
│  │  Scope            │ Entire cluster   │ Selected tables     │    │
│  │  Standby writes   │ Read-only        │ Read-write          │    │
│  │  Version match    │ Required         │ Flexible            │    │
│  │  DDL replication  │ Automatic        │ Manual              │    │
│  │  Performance      │ Lower overhead   │ Higher overhead     │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## High Availability Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Typical HA Setup                                  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Application Layer                          │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐                    │   │
│  │  │   App    │  │   App    │  │   App    │                    │   │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘                    │   │
│  └───────┼─────────────┼─────────────┼───────────────────────────┘   │
│          │             │             │                               │
│          └─────────────┼─────────────┘                               │
│                        ▼                                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │              Connection Pooler (PgBouncer)                    │   │
│  │              Load Balancer / VIP                              │   │
│  └─────────────────────────┬────────────────────────────────────┘   │
│                            │                                         │
│          ┌─────────────────┼─────────────────┐                      │
│          ▼                 ▼                 ▼                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Primary    │  │   Standby    │  │   Standby    │              │
│  │  (Read/Write)│◄─│   (Sync)     │  │   (Async)    │              │
│  │              │──│  (Read-only) │  │  (Read-only) │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│          │                                                           │
│          ▼                                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │            HA Manager (Patroni / pg_auto_failover)            │   │
│  │            Handles failover, leader election                  │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference

```sql
-- Streaming replication status
SELECT * FROM pg_stat_replication;
SELECT * FROM pg_stat_wal_receiver;

-- Logical replication
CREATE PUBLICATION mypub FOR TABLE users, orders;
CREATE SUBSCRIPTION mysub CONNECTION '...' PUBLICATION mypub;

-- Replication slots
SELECT * FROM pg_replication_slots;
SELECT pg_create_physical_replication_slot('slot_name');

-- Backup
pg_basebackup -D /backup -Fp -Xs -P

-- Replication lag
SELECT pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes
FROM pg_stat_replication;
```

---

## Section Files

| File | Topic |
|------|-------|
| [01_streaming_replication.md](01_streaming_replication.md) | Physical replication setup |
| [02_logical_replication.md](02_logical_replication.md) | Publish-subscribe replication |
| [03_high_availability.md](03_high_availability.md) | HA patterns and tools |
| [04_backup_and_recovery.md](04_backup_and_recovery.md) | Backup strategies and PITR |

---

## Further Reading

- PostgreSQL High Availability documentation
- PostgreSQL Streaming Replication
- PostgreSQL Logical Replication
