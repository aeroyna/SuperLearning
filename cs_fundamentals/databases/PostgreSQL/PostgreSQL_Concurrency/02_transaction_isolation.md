# Transaction Isolation Levels

## Learning Objectives
- Understand read phenomena and anomalies
- Master PostgreSQL's isolation levels
- Implement Serializable Snapshot Isolation
- Choose appropriate isolation for use cases

---

## 1. Read Phenomena

### Types of Anomalies

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Read Phenomena                                    │
│                                                                      │
│  1. DIRTY READ                                                       │
│     Reading uncommitted changes from another transaction             │
│     ┌─────────────────────────────────────────────────────────┐     │
│     │ T1: UPDATE balance = 50 (not committed)                 │     │
│     │ T2: SELECT balance → 50 (dirty read!)                   │     │
│     │ T1: ROLLBACK                                             │     │
│     │ T2 read data that never existed!                        │     │
│     └─────────────────────────────────────────────────────────┘     │
│                                                                      │
│  2. NON-REPEATABLE READ                                              │
│     Same query returns different results within transaction          │
│     ┌─────────────────────────────────────────────────────────┐     │
│     │ T1: SELECT balance → 100                                │     │
│     │ T2: UPDATE balance = 50; COMMIT                         │     │
│     │ T1: SELECT balance → 50 (different!)                    │     │
│     └─────────────────────────────────────────────────────────┘     │
│                                                                      │
│  3. PHANTOM READ                                                     │
│     New rows appear in repeated query                                │
│     ┌─────────────────────────────────────────────────────────┐     │
│     │ T1: SELECT * WHERE status='active' → 10 rows            │     │
│     │ T2: INSERT status='active'; COMMIT                      │     │
│     │ T1: SELECT * WHERE status='active' → 11 rows (phantom!) │     │
│     └─────────────────────────────────────────────────────────┘     │
│                                                                      │
│  4. SERIALIZATION ANOMALY                                            │
│     Result differs from any serial execution                         │
│     (Covered in detail below)                                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Write Skew Anomaly

```sql
-- Classic example: On-call doctors
-- Constraint: At least one doctor must be on call

CREATE TABLE doctors (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    on_call BOOLEAN
);

INSERT INTO doctors VALUES (1, 'Alice', true), (2, 'Bob', true);

-- Both doctors try to go off-call simultaneously

-- Transaction 1 (Alice):
BEGIN;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- Returns 2
-- "There's still Bob, I can go off-call"
UPDATE doctors SET on_call = false WHERE id = 1;

-- Transaction 2 (Bob), concurrently:
BEGIN;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- Returns 2
-- "There's still Alice, I can go off-call"
UPDATE doctors SET on_call = false WHERE id = 2;

-- Both commit
COMMIT;  -- T1
COMMIT;  -- T2

-- Result: NO doctors on call! (violated constraint)
-- This is a serialization anomaly (write skew)
```

---

## 2. PostgreSQL Isolation Levels

### Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Isolation Levels in PostgreSQL                    │
│                                                                      │
│  Level              │ Dirty │ Non-Rep │ Phantom │ Serial  │         │
│                     │ Read  │ Read    │ Read    │ Anomaly │         │
│  ───────────────────────────────────────────────────────────────    │
│  Read Uncommitted*  │  No   │  Yes    │  Yes    │  Yes    │         │
│  Read Committed     │  No   │  Yes    │  Yes    │  Yes    │         │
│  Repeatable Read    │  No   │  No     │  No**   │  Yes    │         │
│  Serializable       │  No   │  No     │  No     │  No     │         │
│                                                                      │
│  * PostgreSQL treats Read Uncommitted as Read Committed              │
│  ** PostgreSQL's Repeatable Read prevents phantom reads              │
│     (stronger than SQL standard requires)                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Setting Isolation Level

```sql
-- For single transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Or
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- For session
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check current level
SHOW transaction_isolation;

-- Default level
SET default_transaction_isolation = 'read committed';
```

---

## 3. Read Committed

### Behavior

```sql
-- Default isolation level
-- New snapshot for each statement

-- Session 1:
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100

-- Session 2:
BEGIN;
UPDATE accounts SET balance = 50 WHERE id = 1;
COMMIT;

-- Session 1 (new statement, new snapshot):
SELECT balance FROM accounts WHERE id = 1;  -- Returns 50 (sees commit)
COMMIT;
```

### Read Committed Caveats

```sql
-- UPDATE may see changes mid-statement!

-- Table: users (id, status, updated_at)
-- Rows: (1, 'active', '2024-01-01'), (2, 'active', '2024-01-02')

-- Session 1:
BEGIN;
UPDATE users SET status = 'inactive'
WHERE status = 'active';
-- Locks row 1, starts updating...

-- Session 2 (concurrent):
BEGIN;
UPDATE users SET status = 'pending' WHERE id = 1;
-- Blocks on row 1, waiting for S1

-- Session 1 continues, commits
COMMIT;

-- Session 2 unblocks:
-- Re-evaluates WHERE for row 1
-- Row 1 is now 'inactive', not 'active'
-- Condition may or may not match depending on query
```

### When to Use

```sql
-- Good for:
-- • High-concurrency OLTP with simple queries
-- • Short transactions
-- • When seeing latest committed data is acceptable

-- Not good for:
-- • Complex multi-statement transactions
-- • Reports requiring consistent snapshot
-- • When write skew is a concern
```

---

## 4. Repeatable Read

### Behavior

```sql
-- Snapshot taken at first query in transaction
-- Same snapshot used for entire transaction

-- Session 1:
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
-- Snapshot taken here

-- Session 2:
UPDATE accounts SET balance = 50 WHERE id = 1;
COMMIT;

-- Session 1:
SELECT balance FROM accounts WHERE id = 1;  -- Still returns 100!
-- Same snapshot, doesn't see S2's commit
COMMIT;
```

### Update Conflicts

```sql
-- Repeatable Read prevents lost updates

-- Session 1:
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
UPDATE accounts SET balance = 90 WHERE id = 1;
-- Holds lock on row

-- Session 2:
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 100
UPDATE accounts SET balance = 80 WHERE id = 1;
-- Blocks, waiting for S1

-- Session 1:
COMMIT;

-- Session 2:
-- ERROR: could not serialize access due to concurrent update
-- Must retry transaction!
ROLLBACK;
```

### Write Skew Still Possible

```sql
-- Repeatable Read doesn't prevent write skew!

-- On-call doctors example still fails at Repeatable Read
-- Because each transaction reads different rows than it writes

-- Solution: Use Serializable OR explicit locking
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM doctors WHERE on_call = true FOR UPDATE;
-- Now have exclusive locks on all on-call rows
-- Other transaction must wait
```

---

## 5. Serializable

### Serializable Snapshot Isolation (SSI)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Serializable Snapshot Isolation                   │
│                                                                      │
│  PostgreSQL's Serializable uses SSI (not traditional locking)       │
│                                                                      │
│  How it works:                                                       │
│  1. Runs with Repeatable Read snapshot isolation                     │
│  2. Tracks read/write dependencies between transactions              │
│  3. Detects dangerous patterns that could cause anomalies           │
│  4. Aborts one transaction if serialization failure detected        │
│                                                                      │
│  Benefits:                                                           │
│  • True serializability                                              │
│  • Still uses MVCC (readers don't block)                             │
│  • Only conflicts cause aborts                                       │
│                                                                      │
│  Trade-offs:                                                         │
│  • Some transactions will abort (must retry)                         │
│  • Overhead from dependency tracking                                 │
│  • Memory for predicate locks                                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Write Skew Prevention

```sql
-- On-call doctors with Serializable

-- Session 1 (Alice):
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- Returns 2
UPDATE doctors SET on_call = false WHERE id = 1;

-- Session 2 (Bob):
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT COUNT(*) FROM doctors WHERE on_call = true;  -- Returns 2
UPDATE doctors SET on_call = false WHERE id = 2;

-- Session 1:
COMMIT;  -- Succeeds

-- Session 2:
COMMIT;
-- ERROR: could not serialize access due to read/write dependencies

-- One transaction aborted, constraint preserved!
```

### SSI Configuration

```sql
-- Memory for predicate locks
SHOW max_pred_locks_per_transaction;  -- Default: 64
SHOW max_pred_locks_per_relation;     -- Default: -2 (derived)
SHOW max_pred_locks_per_page;         -- Default: 2

-- Increase for complex transactions
SET max_pred_locks_per_transaction = 128;

-- Monitor SSI conflicts
SELECT * FROM pg_stat_database WHERE datname = current_database();
-- Check: deadlocks, conflicts columns
```

---

## 6. Handling Serialization Failures

### Retry Pattern

```sql
-- Application must retry on serialization failure
-- Error code: 40001 (serialization_failure)

-- Python example:
-- while True:
--     try:
--         with connection.cursor() as cur:
--             cur.execute("BEGIN ISOLATION LEVEL SERIALIZABLE")
--             # ... do work ...
--             cur.execute("COMMIT")
--         break  # Success
--     except psycopg2.errors.SerializationFailure:
--         connection.rollback()
--         # Retry with backoff
--         time.sleep(random.uniform(0.1, 0.5))
```

### PL/pgSQL Retry

```sql
-- Retry within stored procedure
CREATE OR REPLACE FUNCTION transfer_with_retry(
    from_id INT,
    to_id INT,
    amount NUMERIC
) RETURNS VOID AS $$
DECLARE
    retries INT := 0;
    max_retries INT := 5;
BEGIN
    LOOP
        BEGIN
            -- Start serializable transaction
            PERFORM pg_advisory_xact_lock(from_id);
            PERFORM pg_advisory_xact_lock(to_id);

            UPDATE accounts SET balance = balance - amount WHERE id = from_id;
            UPDATE accounts SET balance = balance + amount WHERE id = to_id;

            RETURN;  -- Success

        EXCEPTION
            WHEN serialization_failure OR deadlock_detected THEN
                retries := retries + 1;
                IF retries >= max_retries THEN
                    RAISE;
                END IF;
                PERFORM pg_sleep(random() * 0.1 * retries);
        END;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

---

## 7. Choosing Isolation Level

### Decision Guide

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Isolation Level Selection                         │
│                                                                      │
│  Use READ COMMITTED when:                                            │
│  • Simple, short transactions                                        │
│  • High concurrency requirements                                     │
│  • Each statement can see latest data                               │
│  • No complex consistency requirements                               │
│                                                                      │
│  Use REPEATABLE READ when:                                           │
│  • Need consistent view across multiple statements                   │
│  • Generating reports or analytics                                   │
│  • Can handle retry on update conflicts                              │
│  • Don't have write skew concerns                                    │
│                                                                      │
│  Use SERIALIZABLE when:                                              │
│  • Critical data integrity requirements                              │
│  • Complex invariants involving multiple rows/tables                 │
│  • Write skew must be prevented                                      │
│  • Can implement retry logic                                         │
│  • Performance overhead acceptable                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Performance Comparison

```sql
-- Benchmark different isolation levels
-- Read Committed: Fastest, most concurrent
-- Repeatable Read: Slight overhead for snapshot
-- Serializable: Additional overhead for SSI tracking

-- Check transaction conflicts
SELECT datname,
       xact_commit,
       xact_rollback,
       conflicts
FROM pg_stat_database
WHERE datname = current_database();

-- Monitor for excessive retries
-- High conflict rate = may need different approach
```

---

## 8. Practical Patterns

### Banking Transfer (Serializable)

```sql
-- Ensure consistency for money transfers
BEGIN ISOLATION LEVEL SERIALIZABLE;

-- Check sufficient funds
SELECT balance FROM accounts WHERE id = 1;  -- Source
-- Ensure this is checked atomically with transfer

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

### Report Generation (Repeatable Read)

```sql
-- Consistent snapshot for reporting
BEGIN ISOLATION LEVEL REPEATABLE READ;

-- All these queries see same snapshot
SELECT SUM(balance) FROM accounts;
SELECT COUNT(*) FROM accounts WHERE balance > 1000;
SELECT AVG(balance) FROM accounts GROUP BY account_type;

COMMIT;
```

### High-Throughput Updates (Read Committed + Locking)

```sql
-- When Serializable overhead is too high
-- Use explicit locking with Read Committed

BEGIN;
SELECT * FROM inventory WHERE product_id = 100 FOR UPDATE;
-- Now have exclusive lock

UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 100;
COMMIT;

-- FOR UPDATE SKIP LOCKED for queue-like patterns
SELECT * FROM jobs WHERE status = 'pending'
ORDER BY created_at
LIMIT 1
FOR UPDATE SKIP LOCKED;
```

---

## Summary

| Level | Snapshot | Anomalies Prevented | Use Case |
|-------|----------|---------------------|----------|
| Read Committed | Per statement | Dirty read | High concurrency |
| Repeatable Read | Per transaction | + Non-repeatable, Phantom | Reports, analytics |
| Serializable | Per transaction | + Serialization anomaly | Critical integrity |

---

## Further Reading

- PostgreSQL Transaction Isolation documentation
- Serializable Snapshot Isolation paper
- "Designing Data-Intensive Applications" - Transactions chapter
