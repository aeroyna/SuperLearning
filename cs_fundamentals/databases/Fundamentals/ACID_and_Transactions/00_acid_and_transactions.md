# ACID and Transactions

## Overview

Transactions are fundamental to database reliability. ACID properties ensure that database operations are processed reliably, even in the face of errors, power failures, or concurrent access.

## Topics Covered

1. **[ACID Properties Deep Dive](01_acid_properties_deep_dive.md)** - Atomicity, Consistency, Isolation, Durability explained
2. **[Transaction Lifecycle](02_transaction_lifecycle.md)** - BEGIN, COMMIT, ROLLBACK, and savepoints
3. **[Isolation Levels](03_isolation_levels.md)** - Read phenomena and isolation trade-offs
4. **[Concurrency Control](04_concurrency_control.md)** - How databases handle simultaneous access
5. **[Locking Mechanisms](05_locking_mechanisms.md)** - Pessimistic concurrency with locks
6. **[MVCC Multi-Version Concurrency Control](06_mvcc.md)** - Optimistic concurrency approach

## ACID at a Glance

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ACID PROPERTIES                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A - ATOMICITY                              C - CONSISTENCY                 │
│   ─────────────                              ──────────────                  │
│   "All or Nothing"                           "Valid State to Valid State"   │
│   Either all operations in a                 Transaction takes database     │
│   transaction complete, or                   from one valid state to        │
│   none do.                                   another valid state.           │
│                                                                              │
│   Example: Bank transfer                     Example: Foreign key           │
│   - Debit Account A                          constraints are never          │
│   - Credit Account B                         violated                       │
│   Both succeed or both fail                                                 │
│                                                                              │
│   ───────────────────────────────────────────────────────────────────────   │
│                                                                              │
│   I - ISOLATION                              D - DURABILITY                  │
│   ─────────────                              ─────────────                   │
│   "Concurrent = Serial"                      "Committed = Permanent"        │
│   Concurrent transactions                    Once committed, data           │
│   appear to execute                          survives any subsequent        │
│   serially.                                  failure.                       │
│                                                                              │
│   Example: Two users updating                Example: After COMMIT,         │
│   same row see consistent                    data persists even if          │
│   results                                    server crashes                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Quick Reference

```sql
-- Basic transaction
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- With error handling (PostgreSQL)
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    -- If error occurs...
    SAVEPOINT before_credit;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    -- Can rollback to savepoint
    ROLLBACK TO before_credit;
    -- Or rollback entire transaction
ROLLBACK;

-- Isolation levels
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

## Learning Objectives

After completing this section, you will be able to:
- Explain each ACID property and its guarantees
- Choose appropriate isolation levels for different scenarios
- Understand read phenomena (dirty reads, phantom reads, etc.)
- Implement proper transaction handling in applications
- Debug concurrency issues and deadlocks
