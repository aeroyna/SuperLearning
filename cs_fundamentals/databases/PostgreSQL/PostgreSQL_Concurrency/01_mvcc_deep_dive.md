# MVCC Deep Dive

## Learning Objectives
- Understand Multi-Version Concurrency Control fundamentals
- Learn how PostgreSQL implements tuple versioning
- Master visibility rules and snapshot isolation
- Analyze MVCC overhead and optimization strategies

---

## 1. MVCC Fundamentals

### What is MVCC?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MVCC vs Traditional Locking                       │
│                                                                      │
│  Traditional Locking (Pessimistic):                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Reader acquires shared lock → Blocks writers                │    │
│  │  Writer acquires exclusive lock → Blocks readers & writers   │    │
│  │  Result: Serialized access, lower concurrency                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  MVCC (Optimistic):                                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Keep multiple versions of data                              │    │
│  │  Readers see consistent snapshot (no locks needed)           │    │
│  │  Writers create new versions (don't block readers)           │    │
│  │  Result: High concurrency, readers never blocked             │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  PostgreSQL MVCC Benefits:                                           │
│  • SELECT never blocks (reads from snapshot)                         │
│  • INSERT/UPDATE/DELETE only block conflicting writes               │
│  • No read locks means better scalability                           │
│  • Consistent view of data throughout transaction                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Tuple Versioning

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Tuple Structure                        │
│                                                                      │
│  Each row (tuple) has hidden system columns:                         │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  xmin   │ Transaction ID that inserted this tuple              │  │
│  │  xmax   │ Transaction ID that deleted/updated (0 if live)      │  │
│  │  cmin   │ Command ID within inserting transaction              │  │
│  │  cmax   │ Command ID within deleting transaction               │  │
│  │  ctid   │ Physical location (block, offset)                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Example: UPDATE accounts SET balance = 70 WHERE id = 1             │
│                                                                      │
│  Before (OLD tuple):                                                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ xmin=100 │ xmax=0 │ id=1 │ balance=100                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  After (OLD tuple marked deleted, NEW tuple created):                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ xmin=100 │ xmax=105 │ id=1 │ balance=100   ← Deleted by 105  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ xmin=105 │ xmax=0   │ id=1 │ balance=70    ← New version     │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Viewing System Columns

```sql
-- View hidden columns
SELECT xmin, xmax, ctid, * FROM accounts WHERE id = 1;

-- xmin: Transaction that created this version
-- xmax: Transaction that invalidated this version (0 = still valid)
-- ctid: Physical location (page, offset)

-- Example output:
-- xmin | xmax | ctid  | id | balance
-- -----+------+-------+----+--------
-- 1234 |    0 | (0,1) |  1 |     100

-- Current transaction ID
SELECT txid_current();

-- Transaction snapshot
SELECT txid_current_snapshot();
-- Returns: xmin:xmax:xip_list
-- e.g., 100:105:102,103
-- Meaning: All XIDs < 100 visible, >= 105 invisible,
--          102 and 103 are in-progress (invisible)
```

---

## 2. Visibility Rules

### Tuple Visibility Algorithm

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Visibility Decision Tree                          │
│                                                                      │
│  Is tuple visible to transaction T with snapshot S?                  │
│                                                                      │
│  1. Check xmin (inserting transaction):                              │
│     ├─ xmin is current transaction T → See own changes              │
│     ├─ xmin aborted → Not visible                                    │
│     ├─ xmin in-progress (not T) → Not visible                        │
│     └─ xmin committed AND < S.xmin → Check xmax                      │
│                                                                      │
│  2. Check xmax (deleting transaction):                               │
│     ├─ xmax = 0 → Visible (not deleted)                              │
│     ├─ xmax is current transaction T → Not visible (we deleted it)  │
│     ├─ xmax aborted → Visible (delete was rolled back)               │
│     ├─ xmax in-progress (not T) → Visible (delete not yet committed)│
│     └─ xmax committed AND < S.xmin → Not visible (deleted before S)  │
│                                                                      │
│  Simplified: Tuple visible if:                                       │
│  • Created by committed transaction before our snapshot              │
│  • Not deleted, OR deleted by uncommitted/aborted/future transaction │
└─────────────────────────────────────────────────────────────────────┘
```

### Visibility Examples

```sql
-- Setup
CREATE TABLE test (id INT, value TEXT);
INSERT INTO test VALUES (1, 'original');

-- Session 1: Start transaction
BEGIN;
SELECT txid_current();  -- Returns 1000

-- Session 2: Update the row
BEGIN;
SELECT txid_current();  -- Returns 1001
UPDATE test SET value = 'modified' WHERE id = 1;
COMMIT;

-- Session 1: Still sees original (snapshot isolation)
SELECT * FROM test WHERE id = 1;
-- id | value
----+----------
--  1 | original

-- Because Session 1's snapshot was taken before 1001 committed
COMMIT;

-- Now Session 1 starts new transaction, sees modified
BEGIN;
SELECT * FROM test WHERE id = 1;
-- id | value
----+----------
--  1 | modified
```

---

## 3. Snapshots

### Snapshot Components

```sql
-- Get current snapshot
SELECT txid_current_snapshot();
-- Returns format: xmin:xmax:xip_list
-- Example: 100:110:102,105,107

-- Meaning:
-- xmin (100): All transactions < 100 are complete
-- xmax (110): All transactions >= 110 haven't started
-- xip_list (102,105,107): In-progress transactions between xmin and xmax
```

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Snapshot Visualization                            │
│                                                                      │
│  Snapshot: 100:110:102,105,107                                       │
│                                                                      │
│  Transaction IDs:                                                    │
│  ... 98  99  100 101 102 103 104 105 106 107 108 109 110 111 ...    │
│      ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼   ▼          │
│      C   C   C   C   IP  C   C   IP  C   IP  C   C   F   F          │
│                                                                      │
│  C = Committed (visible if tuple not deleted by C)                   │
│  IP = In-Progress (invisible, in xip_list)                           │
│  F = Future (invisible, >= xmax)                                     │
│                                                                      │
│  Visibility:                                                         │
│  < 100: Committed, visible                                           │
│  100-109: Check if in xip_list; if so, invisible                    │
│  >= 110: Future, invisible                                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Snapshot Types

```sql
-- Read Committed: New snapshot for each statement
BEGIN;
SELECT * FROM accounts;  -- Snapshot at statement start
-- Other transaction commits changes here
SELECT * FROM accounts;  -- NEW snapshot, sees committed changes
COMMIT;

-- Repeatable Read/Serializable: Snapshot at transaction start
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM accounts;  -- Snapshot at transaction start
-- Other transaction commits changes here
SELECT * FROM accounts;  -- SAME snapshot, doesn't see changes
COMMIT;
```

---

## 4. Transaction IDs

### 32-bit Transaction ID

```sql
-- PostgreSQL uses 32-bit unsigned transaction IDs
-- Range: 0 to 2^32-1 (about 4 billion)
-- IDs wrap around (circular comparison)

-- Check current transaction ID
SELECT txid_current();

-- Transaction ID is assigned when first write occurs
BEGIN;
SELECT txid_current();  -- New XID assigned
-- Or when explicitly requested

-- Special XIDs:
-- 0: Invalid XID
-- 1: Bootstrap XID
-- 2: Frozen XID (special, always visible)
```

### Transaction ID Wraparound

```
┌─────────────────────────────────────────────────────────────────────┐
│                    XID Wraparound Problem                            │
│                                                                      │
│  32-bit XIDs eventually wrap around:                                 │
│                                                                      │
│  Time →                                                              │
│  ... 4294967290, 4294967291, 4294967292, 4294967293, 4294967294,    │
│  4294967295, 0, 1, 2, 3, 4, 5 ...                                   │
│                                                                      │
│  Problem: After wraparound, old XIDs appear "in the future"         │
│  A tuple with xmin=100 looks newer than xmin=4294967000             │
│                                                                      │
│  Solution: VACUUM FREEZE                                             │
│  • Old tuples get xmin set to FrozenXID (2)                          │
│  • FrozenXID is always considered "in the past"                     │
│  • Must freeze before XIDs wrap around                               │
│                                                                      │
│  autovacuum_freeze_max_age: Force VACUUM when this old (default 200M)│
│  vacuum_freeze_min_age: Freeze tuples this old (default 50M)        │
└─────────────────────────────────────────────────────────────────────┘
```

```sql
-- Check oldest unfrozen XID
SELECT datname, age(datfrozenxid) AS xid_age
FROM pg_database
ORDER BY age(datfrozenxid) DESC;

-- Check table freeze status
SELECT relname, age(relfrozenxid) AS xid_age
FROM pg_class
WHERE relkind = 'r'
ORDER BY age(relfrozenxid) DESC
LIMIT 10;

-- Manual freeze
VACUUM FREEZE tablename;
```

---

## 5. Commit Log (CLOG)

### Transaction Status Storage

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Commit Log (CLOG)                                 │
│                                                                      │
│  Stores transaction status (2 bits per transaction):                 │
│  00 = In Progress                                                    │
│  01 = Committed                                                      │
│  10 = Aborted                                                        │
│  11 = Sub-committed (subtransaction)                                 │
│                                                                      │
│  Location: $PGDATA/pg_xact/                                          │
│                                                                      │
│  Each CLOG segment: 256KB = covers 1M transactions                   │
│                                                                      │
│  Visibility check:                                                   │
│  1. Check tuple's xmin/xmax                                          │
│  2. Look up CLOG to confirm committed/aborted                        │
│  3. Check against snapshot                                           │
│                                                                      │
│  CLOG is cached in memory for fast access                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Hint Bits

```sql
-- To avoid repeated CLOG lookups, PostgreSQL uses hint bits
-- Stored directly in tuple header after first visibility check

-- Hint bit flags:
-- HEAP_XMIN_COMMITTED: xmin is known committed
-- HEAP_XMIN_INVALID: xmin is known aborted
-- HEAP_XMAX_COMMITTED: xmax is known committed
-- HEAP_XMAX_INVALID: xmax is known aborted

-- First SELECT on tuple:
-- 1. Check CLOG for xmin status
-- 2. Set hint bit in tuple header
-- 3. Future reads skip CLOG lookup

-- Note: Setting hint bits dirties the page
-- This can cause unexpected writes during SELECT
```

---

## 6. MVCC in Action

### Concurrent Transactions Example

```sql
-- Table setup
CREATE TABLE inventory (
    product_id INT PRIMARY KEY,
    quantity INT
);
INSERT INTO inventory VALUES (1, 100);

-- Transaction 1 (T1): XID = 1000
BEGIN;
UPDATE inventory SET quantity = quantity - 10 WHERE product_id = 1;
-- Creates new tuple: xmin=1000, xmax=0, quantity=90
-- Old tuple: xmin=999, xmax=1000

-- Transaction 2 (T2): XID = 1001, before T1 commits
BEGIN;
SELECT quantity FROM inventory WHERE product_id = 1;
-- Sees old tuple (xmin=999 committed, xmax=1000 in-progress)
-- Returns: 100

-- T1 commits
COMMIT;

-- T2 (Read Committed) runs new SELECT
SELECT quantity FROM inventory WHERE product_id = 1;
-- New snapshot sees T1 committed
-- Returns: 90

-- T2 (Repeatable Read) would still see 100
-- Because snapshot taken at transaction start
```

### Write Conflicts

```sql
-- Both try to update same row
-- T1:
BEGIN;
UPDATE inventory SET quantity = 90 WHERE product_id = 1;
-- Holds row lock

-- T2:
BEGIN;
UPDATE inventory SET quantity = 80 WHERE product_id = 1;
-- Blocks! Waits for T1's row lock

-- T1 commits
COMMIT;

-- T2 unblocks, but behavior depends on isolation level:
-- Read Committed: Re-evaluates WHERE, applies update to T1's version
-- Repeatable Read: ERROR: could not serialize access due to concurrent update
-- Serializable: ERROR: could not serialize access due to concurrent update
```

---

## 7. MVCC Overhead

### Dead Tuples

```sql
-- Every UPDATE creates a dead tuple
-- Dead tuples consume space until VACUUMed

-- Check dead tuples
SELECT
    relname,
    n_live_tup,
    n_dead_tup,
    ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Table bloat from dead tuples
SELECT
    schemaname,
    relname,
    pg_size_pretty(pg_relation_size(relid)) AS size
FROM pg_stat_user_tables
ORDER BY pg_relation_size(relid) DESC;
```

### Index Bloat

```sql
-- Indexes also contain entries for dead tuples
-- Until VACUUM marks them dead

-- Check index size vs table size
SELECT
    tablename,
    pg_size_pretty(pg_relation_size(tablename::regclass)) AS table_size,
    pg_size_pretty(pg_indexes_size(tablename::regclass)) AS indexes_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_relation_size(tablename::regclass) DESC;
```

### Snapshot Too Old

```sql
-- Long-running transactions hold back VACUUM
-- Old snapshots prevent tuple cleanup

-- Check oldest transaction
SELECT
    pid,
    age(backend_xid) AS xid_age,
    state,
    query_start,
    NOW() - query_start AS duration,
    LEFT(query, 50) AS query
FROM pg_stat_activity
WHERE backend_xid IS NOT NULL
ORDER BY age(backend_xid) DESC;

-- Terminate long-running transactions if needed
SELECT pg_terminate_backend(pid);
```

---

## 8. Best Practices

### Minimizing MVCC Overhead

```sql
-- 1. Keep transactions short
BEGIN;
-- Do work quickly
COMMIT;

-- 2. Avoid long-running transactions
-- Use statement_timeout and idle_in_transaction_session_timeout
SET statement_timeout = '30s';
SET idle_in_transaction_session_timeout = '5min';

-- 3. Configure autovacuum properly
-- More aggressive for high-update tables
ALTER TABLE high_update_table SET (
    autovacuum_vacuum_scale_factor = 0.05,
    autovacuum_analyze_scale_factor = 0.02
);

-- 4. Monitor dead tuples and bloat
-- Regular checks on pg_stat_user_tables

-- 5. Use HOT updates when possible
-- Keep fillfactor < 100 for updated tables
ALTER TABLE frequently_updated SET (fillfactor = 80);
```

---

## Summary

| Concept | Description |
|---------|-------------|
| MVCC | Multiple row versions for concurrent access |
| xmin/xmax | Transaction IDs for visibility tracking |
| Snapshot | Consistent view of transaction states |
| CLOG | Commit status storage |
| Hint Bits | Cached visibility status in tuples |
| Dead Tuples | Old versions awaiting VACUUM |

---

## Further Reading

- PostgreSQL MVCC documentation
- "PostgreSQL 14 Internals" - Transaction Processing chapter
- MVCC Unmasked (Bruce Momjian presentation)
