# MySQL Replication

## Overview

MySQL replication allows data from one database server (source/master) to be copied to one or more servers (replicas/slaves). This enables horizontal scaling for reads, high availability, and data redundancy.

This section covers:

1. **[Binary Log Replication](01_binary_log_replication.md)** - Core replication mechanism
2. **[Master-Slave Setup](02_master_slave_setup.md)** - Traditional replication configuration
3. **[Group Replication](03_group_replication.md)** - Multi-master synchronous replication
4. **[Replication Lag and Troubleshooting](04_replication_lag_and_troubleshooting.md)** - Monitoring and fixes

---

## Replication Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    MySQL Replication Overview                            │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                     Traditional Replication                      │    │
│  │                                                                  │    │
│  │    ┌──────────┐                                                  │    │
│  │    │  Source  │ ─── Binary Log ───┐                              │    │
│  │    │ (Master) │                   │                              │    │
│  │    └──────────┘                   │                              │    │
│  │                                   ▼                              │    │
│  │         ┌──────────┐    ┌──────────┐    ┌──────────┐            │    │
│  │         │ Replica1 │    │ Replica2 │    │ Replica3 │            │    │
│  │         │ (Slave)  │    │ (Slave)  │    │ (Slave)  │            │    │
│  │         └──────────┘    └──────────┘    └──────────┘            │    │
│  │                                                                  │    │
│  │    Writes: Source only                                           │    │
│  │    Reads: Any server                                             │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                      Group Replication                           │    │
│  │                                                                  │    │
│  │         ┌──────────┐    ┌──────────┐    ┌──────────┐            │    │
│  │         │  Node 1  │◄──►│  Node 2  │◄──►│  Node 3  │            │    │
│  │         │ (Primary)│    │(Secondary│    │(Secondary│            │    │
│  │         └──────────┘    └──────────┘    └──────────┘            │    │
│  │              ▲              ▲              ▲                     │    │
│  │              └──────────────┴──────────────┘                     │    │
│  │                    Group Communication                           │    │
│  │                                                                  │    │
│  │    Writes: Primary (or all in multi-primary)                     │    │
│  │    Reads: Any node                                               │    │
│  │    Auto failover: Yes                                            │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Replication Types

| Type | Description | Use Case |
|------|-------------|----------|
| Async | Replica may lag behind source | Default, read scaling |
| Semi-sync | At least one replica confirms receipt | Better durability |
| Group Replication | Synchronous multi-master | High availability |

---

## Key Concepts

### Binary Log (binlog)
- Records all changes to database
- Basis for replication
- Also used for point-in-time recovery

### Relay Log
- Replica's copy of source binary log
- SQL thread applies relay log events

### GTID (Global Transaction ID)
- Unique identifier for each transaction
- Simplifies failover and replica management

---

## Quick Reference

```sql
-- Check replication status
SHOW REPLICA STATUS\G  -- MySQL 8.0.22+
SHOW SLAVE STATUS\G    -- Legacy

-- Binary log position
SHOW MASTER STATUS;

-- Skip a transaction (GTID)
SET GTID_NEXT = 'uuid:transaction_id';
BEGIN; COMMIT;
SET GTID_NEXT = 'AUTOMATIC';
```

---

## Learning Path

1. Understand binary log fundamentals
2. Set up basic master-slave replication
3. Learn about Group Replication for HA
4. Master monitoring and troubleshooting
