# MVCC Deep Dive

## 1. Introduction

**Multi-Version Concurrency Control (MVCC)** is a concurrency control method that provides high concurrency by maintaining multiple versions of data. Readers don't block writers, and writers don't block readers.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MVCC OVERVIEW                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Traditional Locking:                                                      │
│   • Reader blocks writer                                                   │
│   • Writer blocks reader                                                   │
│   • Lower concurrency                                                      │
│                                                                              │
│   MVCC:                                                                     │
│   • Each write creates a new version                                       │
│   • Readers see consistent snapshot                                        │
│   • Writers work on their own version                                      │
│   • High concurrency                                                       │
│                                                                              │
│   Key Principle:                                                            │
│   "Readers never block writers, writers never block readers"               │
│                                                                              │
│   Used by: PostgreSQL, MySQL/InnoDB, Oracle, SQL Server (RCSI)             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. How MVCC Works

### 2.1 Version Tracking

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VERSION TRACKING                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Each row has metadata for versioning:                                    │
│                                                                              │
│   PostgreSQL:                                                               │
│   • xmin: Transaction ID that created the row                              │
│   • xmax: Transaction ID that deleted/updated the row (0 if active)       │
│   • ctid: Physical location (page, offset)                                 │
│                                                                              │
│   MySQL/InnoDB:                                                             │
│   • DB_TRX_ID: Transaction ID of last modification                         │
│   • DB_ROLL_PTR: Pointer to undo log (previous versions)                   │
│   • DB_ROW_ID: Row ID (if no primary key)                                  │
│                                                                              │
│   Oracle:                                                                   │
│   • SCN (System Change Number) in row header                               │
│   • Undo segments store previous versions                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Visual Example

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MVCC VERSION CHAIN                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Initial state (balance = 1000):                                          │
│   ┌─────────────────────────────────────────────┐                          │
│   │ id=1 │ balance=1000 │ xmin=100 │ xmax=0    │                          │
│   └─────────────────────────────────────────────┘                          │
│                                                                              │
│   Transaction 200 updates balance to 800:                                  │
│   ┌─────────────────────────────────────────────┐                          │
│   │ id=1 │ balance=1000 │ xmin=100 │ xmax=200  │ (old version, "dead")    │
│   └─────────────────────────────────────────────┘                          │
│                           │                                                 │
│                           ▼                                                 │
│   ┌─────────────────────────────────────────────┐                          │
│   │ id=1 │ balance=800  │ xmin=200 │ xmax=0    │ (new version, "live")    │
│   └─────────────────────────────────────────────┘                          │
│                                                                              │
│   Transaction 150 (started before 200) still sees balance=1000            │
│   Transaction 250 (started after 200 committed) sees balance=800          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Snapshots and Visibility

### 3.1 Snapshot Definition

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      TRANSACTION SNAPSHOT                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A snapshot captures:                                                      │
│   • Current transaction ID (my xid)                                        │
│   • List of active (uncommitted) transaction IDs at snapshot time          │
│   • The next transaction ID to be assigned                                 │
│                                                                              │
│   Example snapshot for transaction 150:                                    │
│   {                                                                         │
│     "xid": 150,                                                            │
│     "xmin": 100,           // Oldest active transaction                    │
│     "xmax": 160,           // Next xid to be assigned                     │
│     "active": [105, 120, 148]  // Currently running transactions          │
│   }                                                                         │
│                                                                              │
│   Visibility rules:                                                        │
│   • xmin < snapshot.xmin AND committed → VISIBLE                          │
│   • xmin in snapshot.active → INVISIBLE (not committed yet)               │
│   • xmin >= snapshot.xmax → INVISIBLE (started after snapshot)           │
│   • xmax set and committed before snapshot → INVISIBLE (deleted)          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Visibility Check Algorithm

```python
def is_visible(row, snapshot):
    """
    Determine if a row version is visible to a transaction.

    Args:
        row: Row with xmin, xmax metadata
        snapshot: Transaction's snapshot

    Returns:
        bool: True if row is visible
    """
    # Check row creator (xmin)
    if row.xmin == snapshot.my_xid:
        # Created by this transaction
        if row.xmax == 0:
            return True  # Not deleted
        elif row.xmax == snapshot.my_xid:
            return False  # Deleted by this transaction
        else:
            return True  # Deleted by other (not committed to us)

    if row.xmin >= snapshot.xmax:
        return False  # Created after snapshot

    if row.xmin in snapshot.active:
        return False  # Creator not committed at snapshot time

    if not is_committed(row.xmin):
        return False  # Creator aborted

    # Row was created and committed before snapshot
    # Now check if it was deleted

    if row.xmax == 0:
        return True  # Not deleted

    if row.xmax >= snapshot.xmax:
        return True  # Deleted after snapshot

    if row.xmax in snapshot.active:
        return True  # Deleter not committed at snapshot time

    if not is_committed(row.xmax):
        return True  # Deleter aborted

    return False  # Row was deleted before snapshot
```

---

## 4. PostgreSQL MVCC Implementation

### 4.1 Tuple Structure

```sql
-- View tuple metadata
SELECT
    xmin,           -- Creating transaction
    xmax,           -- Deleting transaction
    cmin,           -- Command ID within creating transaction
    cmax,           -- Command ID within deleting transaction
    ctid,           -- Physical location (page, offset)
    *
FROM accounts;

-- Example output:
--  xmin | xmax | cmin | cmax | ctid  | id | balance
-- ------+------+------+------+-------+----+---------
--   100 |    0 |    0 |    0 | (0,1) |  1 |    1000
--   200 |    0 |    0 |    0 | (0,2) |  2 |    2000
```

### 4.2 PostgreSQL Transaction States

```sql
-- View transaction status
SELECT * FROM pg_stat_activity WHERE state != 'idle';

-- Transaction states:
-- • active: Currently executing
-- • idle in transaction: BEGIN but no current query
-- • idle in transaction (aborted): Error occurred
-- • fastpath function call: Fast-path execution

-- View current snapshot
SELECT pg_current_snapshot();
-- Returns: '100:200:105,148'
-- Meaning: xmin=100, xmax=200, active=[105,148]
```

### 4.3 VACUUM and Dead Tuples

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VACUUM AND DEAD TUPLES                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Updates create new tuple versions; old ones become "dead"                │
│   Dead tuples consume space but are not visible to any transaction         │
│                                                                              │
│   VACUUM process:                                                           │
│   1. Scan table for dead tuples                                            │
│   2. Check if any running transaction might need them                      │
│   3. Mark space as reusable                                                │
│   4. Update visibility map                                                 │
│   5. Update free space map                                                 │
│                                                                              │
│   VACUUM FULL:                                                              │
│   • Rewrites entire table                                                  │
│   • Reclaims space to OS                                                   │
│   • Requires exclusive lock                                                │
│                                                                              │
│   Autovacuum:                                                               │
│   • Background process                                                     │
│   • Runs based on thresholds                                               │
│   • Configure with autovacuum_* parameters                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- Check dead tuples
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    n_dead_tup::float / (n_live_tup + 1) AS dead_ratio,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 0
ORDER BY n_dead_tup DESC;

-- Manual vacuum
VACUUM accounts;              -- Standard vacuum
VACUUM FULL accounts;         -- Aggressive (locks table)
VACUUM ANALYZE accounts;      -- Vacuum + update statistics
```

---

## 5. MySQL/InnoDB MVCC Implementation

### 5.1 Undo Logs

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       INNODB UNDO LOGS                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   InnoDB stores previous versions in UNDO tablespace                       │
│                                                                              │
│   Current Row (in data page):                                              │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │ id=1 │ balance=800 │ TRX_ID=200 │ ROLL_PTR=──────┐         │          │
│   └─────────────────────────────────────────────────────│────────┘          │
│                                                         │                   │
│                                                         ▼                   │
│   Undo Log (previous version):                                             │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │ id=1 │ balance=1000 │ TRX_ID=100 │ ROLL_PTR=NULL           │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                                                              │
│   Multiple updates create a chain of undo records                          │
│   Rollback follows the chain backward                                      │
│   MVCC reads follow chain to find appropriate version                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Read Views

```sql
-- InnoDB creates a "read view" at transaction start (REPEATABLE READ)
-- or at each statement start (READ COMMITTED)

-- Read view contains:
-- • List of active transactions at creation time
-- • Low/high water marks for transaction IDs

-- Check for long-running transactions holding old read views
SELECT
    trx_id,
    trx_started,
    trx_isolation_level,
    trx_rows_locked,
    trx_rows_modified
FROM information_schema.innodb_trx;
```

### 5.3 Purge Process

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      INNODB PURGE                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Purge thread removes old undo log records that are no longer needed      │
│                                                                              │
│   A version can be purged when:                                             │
│   • No active transaction might need it                                    │
│   • All transactions that started before the version was created           │
│     have completed                                                          │
│                                                                              │
│   Purge lag problems:                                                       │
│   • Long-running transactions prevent purge                                │
│   • Undo tablespace grows                                                  │
│   • Query performance degrades                                             │
│                                                                              │
│   Monitor purge:                                                            │
│   SHOW ENGINE INNODB STATUS;                                               │
│   Look for "History list length" - high values indicate purge lag          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- Monitor undo tablespace
SELECT
    TABLESPACE_NAME,
    FILE_NAME,
    INITIAL_SIZE,
    AUTOEXTEND_SIZE
FROM information_schema.files
WHERE TABLESPACE_NAME LIKE '%undo%';

-- Check history list length (purge lag)
SHOW ENGINE INNODB STATUS\G
-- Look for: "History list length: XXXXX"
```

---

## 6. MVCC and Isolation Levels

### 6.1 Read Committed with MVCC

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                READ COMMITTED + MVCC                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Each STATEMENT gets a fresh snapshot                                     │
│                                                                              │
│   Transaction 1                     Transaction 2                           │
│   ─────────────                     ─────────────                           │
│   BEGIN                             BEGIN                                   │
│   SELECT balance FROM accounts      UPDATE accounts SET balance=500        │
│   WHERE id=1;                       WHERE id=1;                             │
│   -- Returns 1000                   COMMIT                                  │
│   -- (new snapshot)                                                         │
│   SELECT balance FROM accounts                                              │
│   WHERE id=1;                                                               │
│   -- Returns 500                                                            │
│   -- (got new snapshot, sees T2's commit)                                  │
│   COMMIT                                                                    │
│                                                                              │
│   Non-repeatable reads possible (different values for same query)          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Repeatable Read with MVCC

```
┌─────────────────────────────────────────────────────────────────────────────┐
│               REPEATABLE READ + MVCC                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Single snapshot for entire TRANSACTION                                   │
│                                                                              │
│   Transaction 1                     Transaction 2                           │
│   ─────────────                     ─────────────                           │
│   BEGIN                             BEGIN                                   │
│   SELECT balance FROM accounts      UPDATE accounts SET balance=500        │
│   WHERE id=1;                       WHERE id=1;                             │
│   -- Returns 1000                   COMMIT                                  │
│   -- (snapshot taken at first query)                                       │
│   SELECT balance FROM accounts                                              │
│   WHERE id=1;                                                               │
│   -- Returns 1000                                                           │
│   -- (still using same snapshot)                                           │
│   COMMIT                                                                    │
│                                                                              │
│   Same snapshot used throughout = consistent view                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.3 Serializable with MVCC

```
┌─────────────────────────────────────────────────────────────────────────────┐
│               SERIALIZABLE + MVCC (PostgreSQL SSI)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PostgreSQL uses Serializable Snapshot Isolation (SSI)                    │
│                                                                              │
│   • Uses MVCC for reads (no blocking)                                      │
│   • Tracks read/write dependencies                                         │
│   • Detects serialization anomalies at commit                              │
│   • Aborts one transaction if conflict detected                            │
│                                                                              │
│   Transaction 1                     Transaction 2                           │
│   ─────────────                     ─────────────                           │
│   BEGIN ISOLATION LEVEL             BEGIN ISOLATION LEVEL                   │
│   SERIALIZABLE;                     SERIALIZABLE;                           │
│   SELECT SUM(balance) FROM accounts;  -- T1 reads all                      │
│   -- Returns 5000                                                           │
│                                     INSERT INTO accounts (balance)         │
│                                     VALUES (1000);                          │
│   INSERT INTO accounts (balance)                                            │
│   VALUES (500);                                                             │
│   COMMIT;                           COMMIT;                                 │
│   -- One transaction may fail with serialization error                     │
│                                                                              │
│   Application must retry failed transactions                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Write-Write Conflicts

### 7.1 Lost Update Prevention

```sql
-- MVCC doesn't prevent all write conflicts by itself
-- Still need locking for write-write conflicts

-- Transaction 1                    -- Transaction 2
BEGIN;                              BEGIN;
SELECT balance FROM accounts        SELECT balance FROM accounts
WHERE id = 1;                       WHERE id = 1;
-- Returns 1000                     -- Returns 1000

UPDATE accounts                     UPDATE accounts
SET balance = 1100                  SET balance = 950
WHERE id = 1;                       WHERE id = 1;
                                    -- BLOCKED! T1 holds row lock

COMMIT;                             -- Now T2 can proceed
                                    -- But T2 overwrites T1's update!
                                    COMMIT;

-- Final balance: 950 (T1's update lost)

-- Solution: Use SELECT FOR UPDATE
BEGIN;
SELECT balance FROM accounts
WHERE id = 1 FOR UPDATE;  -- Acquires exclusive lock
-- Other transactions must wait
```

### 7.2 PostgreSQL Write Conflict Handling

```sql
-- In REPEATABLE READ, PostgreSQL detects write conflicts
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE accounts SET balance = 1100 WHERE id = 1;
-- If another transaction already modified this row and committed
-- since our snapshot, we get:
-- ERROR: could not serialize access due to concurrent update

-- Application must handle this:
-- 1. Retry the transaction
-- 2. Use explicit locking (FOR UPDATE)
-- 3. Accept the error
```

---

## 8. MVCC Performance Considerations

### 8.1 Benefits

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MVCC BENEFITS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ✓ High read concurrency                                                  │
│     • Readers never wait for writers                                       │
│     • Writers never wait for readers                                       │
│     • Only write-write conflicts cause waiting                             │
│                                                                              │
│   ✓ Consistent snapshots                                                   │
│     • Queries see stable data                                              │
│     • No dirty reads ever                                                  │
│     • Reports show consistent state                                        │
│                                                                              │
│   ✓ No lock escalation issues                                              │
│     • Read operations don't acquire traditional locks                      │
│     • Reduces deadlock probability                                         │
│                                                                              │
│   ✓ Long-running queries don't block                                       │
│     • Analytics queries can run alongside OLTP                             │
│     • Reporting doesn't impact transactions                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Costs

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       MVCC COSTS                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ✗ Storage overhead                                                        │
│     • Multiple versions consume space                                      │
│     • PostgreSQL: Extra tuple headers (23 bytes each)                      │
│     • InnoDB: Undo log space                                               │
│                                                                              │
│   ✗ Write amplification                                                     │
│     • Updates create new versions (not in-place)                           │
│     • More I/O for updates                                                 │
│     • Index maintenance for new versions                                   │
│                                                                              │
│   ✗ Maintenance overhead                                                    │
│     • Need vacuum/purge processes                                          │
│     • Long-running transactions prevent cleanup                            │
│     • Bloat if maintenance falls behind                                    │
│                                                                              │
│   ✗ Memory usage                                                            │
│     • Snapshot tracking                                                    │
│     • Version chain traversal                                              │
│     • Transaction state maintenance                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.3 Optimization Tips

```sql
-- PostgreSQL: Prevent bloat
-- Keep transactions short
-- Monitor and tune autovacuum

-- Check for long-running transactions
SELECT
    pid,
    age(now(), xact_start) AS transaction_age,
    state,
    query
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
ORDER BY xact_start;

-- Kill long-running transactions if needed
SELECT pg_terminate_backend(pid);

-- MySQL: Monitor undo space
SHOW ENGINE INNODB STATUS;
-- Watch "History list length"

-- Avoid long-running transactions
SET SESSION transaction_isolation = 'READ-COMMITTED';
-- Releases read view after each statement
```

---

## 9. Summary

| Aspect | PostgreSQL | MySQL/InnoDB |
|--------|------------|--------------|
| Version Storage | In table (dead tuples) | Undo tablespace |
| Cleanup | VACUUM/Autovacuum | Purge thread |
| Metadata | xmin, xmax | TRX_ID, ROLL_PTR |
| Snapshot | pg_snapshot | Read view |
| Default Isolation | READ COMMITTED | REPEATABLE READ |

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KEY TAKEAWAYS                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. MVCC provides non-blocking reads through versioning                   │
│                                                                              │
│   2. Each transaction sees a consistent snapshot of the database           │
│                                                                              │
│   3. Old versions must be cleaned up (vacuum/purge)                        │
│                                                                              │
│   4. Long-running transactions prevent cleanup = bloat                     │
│                                                                              │
│   5. Write-write conflicts still require locking                           │
│                                                                              │
│   6. Different isolation levels = different snapshot behavior              │
│                                                                              │
│   7. Monitor vacuum/purge lag to maintain performance                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```
