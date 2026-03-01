# Lock Types and Deadlocks

## Learning Objectives
- Understand PostgreSQL's locking hierarchy
- Master table-level and row-level locks
- Use advisory locks for application logic
- Detect and resolve deadlocks

---

## 1. Lock Overview

### Lock Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Lock Hierarchy                         │
│                                                                      │
│  Object Level Locks:                                                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Database locks                                              │    │
│  │  Schema locks                                                │    │
│  │  Relation (table) locks ← Most common                       │    │
│  │  Page locks (rare, used internally)                          │    │
│  │  Tuple (row) locks                                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Lock Modes (conflict matrix determines compatibility):              │
│                                                                      │
│  Table Locks (8 modes, from weakest to strongest):                  │
│  1. ACCESS SHARE        (SELECT)                                     │
│  2. ROW SHARE           (SELECT FOR UPDATE/SHARE)                    │
│  3. ROW EXCLUSIVE       (INSERT/UPDATE/DELETE)                       │
│  4. SHARE UPDATE EXCLUSIVE (VACUUM, CREATE INDEX CONCURRENTLY)      │
│  5. SHARE               (CREATE INDEX)                               │
│  6. SHARE ROW EXCLUSIVE (deprecated)                                 │
│  7. EXCLUSIVE           (rare)                                       │
│  8. ACCESS EXCLUSIVE    (ALTER TABLE, DROP, VACUUM FULL)             │
│                                                                      │
│  Row Locks (4 modes):                                                │
│  1. FOR KEY SHARE       (foreign key checks)                         │
│  2. FOR SHARE           (SELECT FOR SHARE)                           │
│  3. FOR NO KEY UPDATE   (UPDATE not touching key)                    │
│  4. FOR UPDATE          (SELECT FOR UPDATE, UPDATE, DELETE)          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Table-Level Locks

### Lock Conflict Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Table Lock Conflict Matrix                                │
│                                                                              │
│  Requested    │ AS  RS  RE  SUE  S  SRE  E   AE                             │
│  ─────────────────────────────────────────────────────────────               │
│  ACCESS SHARE │  ·   ·   ·   ·   ·   ·   ·   X                              │
│  ROW SHARE    │  ·   ·   ·   ·   ·   ·   X   X                              │
│  ROW EXCL     │  ·   ·   ·   ·   X   X   X   X                              │
│  SHARE UPD EX │  ·   ·   ·   X   X   X   X   X                              │
│  SHARE        │  ·   ·   X   X   ·   X   X   X                              │
│  SHARE ROW EX │  ·   ·   X   X   X   X   X   X                              │
│  EXCLUSIVE    │  ·   X   X   X   X   X   X   X                              │
│  ACCESS EXCL  │  X   X   X   X   X   X   X   X                              │
│                                                                              │
│  · = Compatible (can be held simultaneously)                                 │
│  X = Conflicts (must wait)                                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Automatic Table Locks

```sql
-- SELECT acquires ACCESS SHARE
SELECT * FROM users;
-- Conflicts only with ACCESS EXCLUSIVE (ALTER TABLE, DROP)

-- INSERT/UPDATE/DELETE acquire ROW EXCLUSIVE
INSERT INTO users VALUES (1, 'John');
UPDATE users SET name = 'Jane' WHERE id = 1;
DELETE FROM users WHERE id = 1;
-- Allows concurrent SELECT, INSERT, UPDATE, DELETE
-- Blocks SHARE, ACCESS EXCLUSIVE

-- CREATE INDEX acquires SHARE lock
CREATE INDEX idx_users_name ON users(name);
-- Blocks INSERT/UPDATE/DELETE until complete

-- CREATE INDEX CONCURRENTLY acquires SHARE UPDATE EXCLUSIVE
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
-- Allows concurrent writes (slower index build)

-- ALTER TABLE acquires ACCESS EXCLUSIVE
ALTER TABLE users ADD COLUMN age INT;
-- Blocks everything including SELECT
```

### Explicit Table Locks

```sql
-- Lock entire table
BEGIN;
LOCK TABLE accounts IN EXCLUSIVE MODE;
-- No other transaction can read or write

-- Common lock modes for explicit locking
LOCK TABLE t IN ACCESS SHARE MODE;           -- Read-only access
LOCK TABLE t IN ROW SHARE MODE;              -- Allow reads, prevent exclusive
LOCK TABLE t IN ROW EXCLUSIVE MODE;          -- Standard write lock
LOCK TABLE t IN SHARE UPDATE EXCLUSIVE MODE; -- Block other maintenance
LOCK TABLE t IN SHARE MODE;                  -- Allow reads, block writes
LOCK TABLE t IN EXCLUSIVE MODE;              -- Block reads and writes
LOCK TABLE t IN ACCESS EXCLUSIVE MODE;       -- Block everything

-- NOWAIT option
LOCK TABLE accounts IN EXCLUSIVE MODE NOWAIT;
-- Fails immediately if lock not available (no waiting)
```

---

## 3. Row-Level Locks

### SELECT FOR UPDATE/SHARE

```sql
-- FOR UPDATE: Exclusive row lock
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Locks row(s) for update
-- Other transactions must wait to update or lock

-- FOR SHARE: Shared row lock
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
-- Multiple transactions can hold FOR SHARE
-- Blocks FOR UPDATE

-- FOR NO KEY UPDATE: Weaker than FOR UPDATE
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR NO KEY UPDATE;
-- Allows FOR KEY SHARE (foreign key checks)
-- Used when not updating primary/unique key columns

-- FOR KEY SHARE: Weakest row lock
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR KEY SHARE;
-- Used by foreign key checks
-- Compatible with FOR NO KEY UPDATE
```

### Lock Options

```sql
-- NOWAIT: Fail immediately if locked
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- ERROR: could not obtain lock on row

-- SKIP LOCKED: Skip locked rows (great for queues)
SELECT * FROM jobs WHERE status = 'pending'
ORDER BY created_at
LIMIT 5
FOR UPDATE SKIP LOCKED;
-- Returns up to 5 unlocked pending jobs

-- OF table_name: Lock specific table in joins
SELECT * FROM orders o
JOIN order_items i ON o.id = i.order_id
WHERE o.id = 100
FOR UPDATE OF orders;
-- Only locks rows in orders table, not order_items
```

### Row Lock Patterns

```sql
-- Queue processing with SKIP LOCKED
CREATE TABLE job_queue (
    id SERIAL PRIMARY KEY,
    payload JSONB,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Worker claims a job
BEGIN;
UPDATE job_queue
SET status = 'processing'
WHERE id = (
    SELECT id FROM job_queue
    WHERE status = 'pending'
    ORDER BY created_at
    LIMIT 1
    FOR UPDATE SKIP LOCKED
)
RETURNING *;
-- Process job...
COMMIT;

-- Multiple workers can claim different jobs concurrently
```

---

## 4. Advisory Locks

### What are Advisory Locks?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Advisory Locks                                    │
│                                                                      │
│  Application-controlled locks (not tied to database objects)        │
│                                                                      │
│  Use cases:                                                          │
│  • Coordinate between application instances                          │
│  • Implement distributed mutex                                       │
│  • Rate limiting                                                     │
│  • Ensure single worker processes job                                │
│                                                                      │
│  Types:                                                              │
│  • Session-level: Held until explicit release or disconnect         │
│  • Transaction-level: Released at transaction end                   │
│                                                                      │
│  Lock key: One or two 32-bit integers (or single 64-bit)            │
└─────────────────────────────────────────────────────────────────────┘
```

### Advisory Lock Functions

```sql
-- Session-level locks (explicit release required)
SELECT pg_advisory_lock(12345);           -- Wait for lock
SELECT pg_advisory_lock(1, 2);            -- Two-int key
SELECT pg_try_advisory_lock(12345);       -- Non-blocking, returns boolean
SELECT pg_advisory_unlock(12345);         -- Release lock
SELECT pg_advisory_unlock_all();          -- Release all session locks

-- Transaction-level locks (auto-release at commit/rollback)
SELECT pg_advisory_xact_lock(12345);      -- Wait for lock
SELECT pg_try_advisory_xact_lock(12345);  -- Non-blocking

-- Shared locks (multiple holders allowed)
SELECT pg_advisory_lock_shared(12345);    -- Session-level shared
SELECT pg_advisory_xact_lock_shared(12345); -- Transaction-level shared
```

### Advisory Lock Patterns

```sql
-- Singleton process (only one instance runs)
CREATE OR REPLACE FUNCTION run_singleton_job()
RETURNS VOID AS $$
BEGIN
    -- Try to acquire lock without waiting
    IF NOT pg_try_advisory_lock(hash_text('singleton_job')) THEN
        RAISE NOTICE 'Job already running';
        RETURN;
    END IF;

    -- Do the job...
    PERFORM pg_sleep(60);  -- Simulate work

    -- Release lock
    PERFORM pg_advisory_unlock(hash_text('singleton_job'));
END;
$$ LANGUAGE plpgsql;

-- User rate limiting
CREATE OR REPLACE FUNCTION check_rate_limit(user_id INT)
RETURNS BOOLEAN AS $$
BEGIN
    -- Try to get lock for this user
    IF pg_try_advisory_xact_lock(user_id) THEN
        RETURN TRUE;  -- Proceed
    ELSE
        RETURN FALSE; -- Rate limited
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Ensure unique processing of entity
BEGIN;
SELECT pg_advisory_xact_lock(hashtext('order_' || order_id::text));
-- Process order, knowing no other connection is processing same order
COMMIT;
```

### Viewing Advisory Locks

```sql
-- Check held advisory locks
SELECT * FROM pg_locks WHERE locktype = 'advisory';

-- Detailed view
SELECT l.pid,
       a.usename,
       l.classid,
       l.objid,
       l.granted,
       a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.locktype = 'advisory';
```

---

## 5. Deadlocks

### How Deadlocks Occur

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Deadlock Example                                  │
│                                                                      │
│  Transaction 1            Transaction 2                              │
│  ─────────────            ─────────────                              │
│  BEGIN;                   BEGIN;                                     │
│  UPDATE t SET x=1         UPDATE t SET x=2                           │
│  WHERE id=1;              WHERE id=2;                                │
│  (holds lock on row 1)    (holds lock on row 2)                      │
│       │                        │                                     │
│       │                        │                                     │
│       ▼                        ▼                                     │
│  UPDATE t SET x=1         UPDATE t SET x=2                           │
│  WHERE id=2;              WHERE id=1;                                │
│  (waits for row 2)        (waits for row 1)                          │
│       │                        │                                     │
│       └────────────────────────┘                                     │
│                  │                                                   │
│                  ▼                                                   │
│           DEADLOCK!                                                  │
│                                                                      │
│  PostgreSQL detects and aborts one transaction (T2):                 │
│  ERROR: deadlock detected                                            │
│  DETAIL: Process 1234 waits for ShareLock on transaction 5678;      │
│  Process 5678 waits for ShareLock on transaction 1234.              │
└─────────────────────────────────────────────────────────────────────┘
```

### Deadlock Detection

```sql
-- PostgreSQL checks for deadlocks periodically
SHOW deadlock_timeout;  -- Default: 1s

-- Lower for faster detection (more CPU)
SET deadlock_timeout = '500ms';

-- View deadlock information in logs
-- postgresql.conf:
-- log_lock_waits = on
-- deadlock_timeout = 1s

-- When deadlock_timeout is exceeded, PostgreSQL:
-- 1. Checks for deadlock cycle
-- 2. Chooses victim transaction
-- 3. Aborts victim with error
```

### Preventing Deadlocks

```sql
-- Strategy 1: Lock in consistent order
-- Always lock lower ID first
BEGIN;
SELECT * FROM accounts WHERE id IN (1, 5, 3)
ORDER BY id
FOR UPDATE;
-- Locks row 1, then 3, then 5

-- Strategy 2: Lock all needed rows at once
BEGIN;
SELECT * FROM accounts WHERE id = ANY(ARRAY[1, 3, 5])
FOR UPDATE;

-- Strategy 3: Use advisory locks for ordering
BEGIN;
SELECT pg_advisory_xact_lock(LEAST(1, 5));
SELECT pg_advisory_xact_lock(GREATEST(1, 5));
-- Now lock actual rows

-- Strategy 4: Use NOWAIT or SKIP LOCKED
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- Fails immediately if locked, no deadlock possible

-- Strategy 5: Keep transactions short
-- Reduces window for conflicts
```

---

## 6. Monitoring Locks

### pg_locks View

```sql
-- All current locks
SELECT * FROM pg_locks;

-- Locks with details
SELECT
    l.locktype,
    l.relation::regclass AS table_name,
    l.mode,
    l.granted,
    l.pid,
    a.usename,
    a.query_start,
    a.state,
    LEFT(a.query, 60) AS query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.relation IS NOT NULL
ORDER BY l.relation, l.mode;

-- Waiting locks (blocked queries)
SELECT
    blocked.pid AS blocked_pid,
    blocked.usename AS blocked_user,
    blocking.pid AS blocking_pid,
    blocking.usename AS blocking_user,
    blocked.query AS blocked_query,
    blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_locks blocked_locks ON blocked.pid = blocked_locks.pid
JOIN pg_locks blocking_locks ON blocked_locks.locktype = blocking_locks.locktype
    AND blocked_locks.relation = blocking_locks.relation
    AND blocked_locks.pid != blocking_locks.pid
JOIN pg_stat_activity blocking ON blocking_locks.pid = blocking.pid
WHERE NOT blocked_locks.granted;
```

### Lock Wait Analysis

```sql
-- Who is blocking whom (PostgreSQL 9.6+)
SELECT * FROM pg_blocking_pids(12345);  -- PIDs blocking process 12345

-- Detailed blocking information (PostgreSQL 14+)
SELECT
    wait.pid AS waiting_pid,
    wait.wait_event_type,
    wait.wait_event,
    block.pid AS blocking_pid,
    block.state AS blocking_state
FROM pg_stat_activity wait
JOIN pg_stat_activity block ON block.pid = ANY(pg_blocking_pids(wait.pid))
WHERE wait.state = 'active'
  AND wait.wait_event_type = 'Lock';

-- Terminate blocking connection
SELECT pg_terminate_backend(blocking_pid);
-- Or cancel query
SELECT pg_cancel_backend(blocking_pid);
```

### Lock Timeout

```sql
-- Limit time waiting for locks
SET lock_timeout = '5s';

BEGIN;
SELECT * FROM accounts FOR UPDATE;
-- Will abort after 5s if cannot acquire lock
-- ERROR: canceling statement due to lock timeout

-- Per-statement timeout
SET LOCAL lock_timeout = '3s';

-- Useful to prevent queries from hanging indefinitely
```

---

## 7. Practical Examples

### Safe Balance Transfer

```sql
-- Consistent ordering prevents deadlocks
CREATE OR REPLACE FUNCTION transfer(
    from_account INT,
    to_account INT,
    amount NUMERIC
) RETURNS VOID AS $$
BEGIN
    -- Lock accounts in consistent order (by ID)
    PERFORM 1 FROM accounts
    WHERE id IN (from_account, to_account)
    ORDER BY id
    FOR UPDATE;

    -- Check sufficient funds
    IF (SELECT balance FROM accounts WHERE id = from_account) < amount THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;

    -- Perform transfer
    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;
END;
$$ LANGUAGE plpgsql;
```

### Batch Processing with Skip Locked

```sql
-- Process items in batches without blocking
CREATE OR REPLACE FUNCTION process_batch(batch_size INT)
RETURNS TABLE (processed_id INT) AS $$
BEGIN
    RETURN QUERY
    UPDATE items
    SET status = 'processing',
        processed_at = NOW()
    WHERE id IN (
        SELECT id FROM items
        WHERE status = 'pending'
        ORDER BY created_at
        LIMIT batch_size
        FOR UPDATE SKIP LOCKED
    )
    RETURNING id;
END;
$$ LANGUAGE plpgsql;

-- Multiple workers can call this concurrently
SELECT * FROM process_batch(100);
```

---

## Summary

| Lock Type | Scope | Use Case |
|-----------|-------|----------|
| Table Locks | Entire table | DDL, bulk operations |
| Row Locks | Individual rows | DML, SELECT FOR UPDATE |
| Advisory Locks | Application-defined | Custom synchronization |

---

## Further Reading

- PostgreSQL Explicit Locking documentation
- Lock Management Functions
- MVCC and Locking internals
