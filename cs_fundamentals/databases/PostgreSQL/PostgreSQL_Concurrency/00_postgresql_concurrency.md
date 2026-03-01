# PostgreSQL Concurrency

## Overview

PostgreSQL uses sophisticated concurrency control mechanisms to allow multiple transactions to access data simultaneously while maintaining consistency. Understanding these mechanisms is essential for building reliable, high-performance database applications.

---

## What You'll Learn

### 1. MVCC Deep Dive
- Multi-Version Concurrency Control principles
- Tuple versioning and visibility
- Snapshot isolation mechanics
- MVCC overhead and trade-offs

### 2. Transaction Isolation Levels
- Read phenomena (dirty reads, phantom reads, etc.)
- PostgreSQL isolation levels
- Serializable Snapshot Isolation (SSI)
- Choosing the right isolation level

### 3. Lock Types and Deadlocks
- Table-level locks
- Row-level locks
- Advisory locks
- Deadlock detection and resolution

### 4. Vacuum and Bloat Management
- Why VACUUM is necessary
- Autovacuum configuration
- Table bloat detection and remediation
- VACUUM FULL vs regular VACUUM

---

## Why Concurrency Matters

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Concurrency Challenges                            │
│                                                                      │
│  Without Concurrency Control:                                        │
│                                                                      │
│  Transaction A                    Transaction B                      │
│  ─────────────                    ─────────────                      │
│  READ balance: $100                                                  │
│                                   READ balance: $100                 │
│  SET balance = $100 - $30                                            │
│                                   SET balance = $100 - $50           │
│  COMMIT (balance = $70)                                              │
│                                   COMMIT (balance = $50)             │
│                                                                      │
│  Lost Update! $30 withdrawal disappeared.                           │
│  Expected: $20 (100 - 30 - 50)                                       │
│  Actual: $50                                                         │
│                                                                      │
│  PostgreSQL's MVCC prevents this while allowing concurrent access   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts

### MVCC (Multi-Version Concurrency Control)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MVCC Principle                                    │
│                                                                      │
│  "Readers don't block writers, writers don't block readers"         │
│                                                                      │
│  Instead of locking data:                                            │
│  • Keep multiple versions of each row                                │
│  • Each transaction sees a consistent snapshot                       │
│  • Writers create new versions                                       │
│  • Readers see appropriate version based on snapshot                 │
│                                                                      │
│  Row Versions:                                                       │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  Version 1: balance=$100 (xmin=100, xmax=105)              │     │
│  │  Version 2: balance=$70  (xmin=105, xmax=∞)                │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Transaction 102 (started before 105):                               │
│  → Sees Version 1 (balance=$100)                                    │
│                                                                      │
│  Transaction 110 (started after 105 committed):                      │
│  → Sees Version 2 (balance=$70)                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Transaction Isolation

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Serialization Anomaly |
|-------|------------|---------------------|--------------|----------------------|
| Read Uncommitted | Possible* | Possible | Possible | Possible |
| Read Committed | No | Possible | Possible | Possible |
| Repeatable Read | No | No | No** | Possible |
| Serializable | No | No | No | No |

*PostgreSQL treats Read Uncommitted as Read Committed
**PostgreSQL's Repeatable Read prevents phantom reads

### Lock Hierarchy

```
Table Locks (weakest to strongest):
ACCESS SHARE < ROW SHARE < ROW EXCLUSIVE < SHARE UPDATE EXCLUSIVE
< SHARE < SHARE ROW EXCLUSIVE < EXCLUSIVE < ACCESS EXCLUSIVE

Row Locks:
FOR KEY SHARE < FOR SHARE < FOR NO KEY UPDATE < FOR UPDATE
```

---

## Section Files

| File | Topic |
|------|-------|
| [01_mvcc_deep_dive.md](01_mvcc_deep_dive.md) | MVCC internals and visibility |
| [02_transaction_isolation.md](02_transaction_isolation.md) | Isolation levels and anomalies |
| [03_locks_and_deadlocks.md](03_locks_and_deadlocks.md) | Locking mechanisms |
| [04_vacuum_and_bloat.md](04_vacuum_and_bloat.md) | Maintenance and bloat |

---

## Quick Reference

```sql
-- View current transactions
SELECT * FROM pg_stat_activity WHERE state = 'active';

-- Check locks
SELECT * FROM pg_locks WHERE NOT granted;

-- Transaction isolation
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN ISOLATION LEVEL REPEATABLE READ;

-- Explicit locking
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
LOCK TABLE accounts IN EXCLUSIVE MODE;

-- Vacuum operations
VACUUM ANALYZE tablename;
VACUUM FULL tablename;
```

---

## Further Reading

- PostgreSQL Concurrency Control documentation
- PostgreSQL Transaction Isolation
- "PostgreSQL 14 Internals" - MVCC chapter
