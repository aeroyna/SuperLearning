# Locking Mechanisms

## 1. Introduction

**Locking** is a mechanism to control concurrent access to database resources. It prevents multiple transactions from modifying data in ways that cause anomalies.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     LOCKING OVERVIEW                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Locks control:                                                            │
│   • WHO can access data (which transaction)                                │
│   • WHAT operations are allowed (read/write)                               │
│   • WHEN access is permitted (lock granted/waiting)                        │
│   • HOW MUCH data is locked (granularity)                                  │
│                                                                              │
│   Lock Manager responsibilities:                                            │
│   • Maintain lock table                                                    │
│   • Grant/deny lock requests                                               │
│   • Detect and resolve deadlocks                                           │
│   • Release locks on transaction end                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Lock Types

### 2.1 Basic Lock Modes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      BASIC LOCK MODES                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SHARED LOCK (S)                                                           │
│   ─────────────                                                             │
│   • For reading data                                                        │
│   • Multiple transactions can hold simultaneously                          │
│   • Blocks exclusive locks                                                 │
│   • Released after read (or at commit in strict 2PL)                       │
│                                                                              │
│   EXCLUSIVE LOCK (X)                                                        │
│   ──────────────                                                            │
│   • For writing data                                                        │
│   • Only one transaction can hold                                          │
│   • Blocks all other locks                                                 │
│   • Held until transaction ends                                            │
│                                                                              │
│   UPDATE LOCK (U) - SQL Server                                              │
│   ────────────────────────────                                              │
│   • For "read-then-update" pattern                                         │
│   • Compatible with S locks but not other U locks                          │
│   • Converts to X lock when update happens                                 │
│   • Prevents conversion deadlocks                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Lock Compatibility Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   LOCK COMPATIBILITY MATRIX                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Requested    │ Existing Lock                                              │
│                │    None    │  S (Shared) │  U (Update) │  X (Exclusive)   │
│   ─────────────┼────────────┼─────────────┼─────────────┼──────────────────│
│   S (Shared)   │    ✓       │      ✓      │      ✓      │       ✗         │
│   U (Update)   │    ✓       │      ✓      │      ✗      │       ✗         │
│   X (Exclusive)│    ✓       │      ✗      │      ✗      │       ✗         │
│                                                                              │
│   ✓ = Compatible (lock granted)                                            │
│   ✗ = Conflict (must wait or fail)                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Intent Locks

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       INTENT LOCKS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Intent locks signal intention to lock at a finer granularity.            │
│                                                                              │
│   IS (Intent Shared)                                                        │
│   • "I intend to acquire S locks on rows in this table"                    │
│   • Compatible with other IS and IX on table                               │
│                                                                              │
│   IX (Intent Exclusive)                                                     │
│   • "I intend to acquire X locks on rows in this table"                    │
│   • Blocks table-level S and X locks                                       │
│                                                                              │
│   SIX (Shared + Intent Exclusive)                                          │
│   • "I'm reading the whole table but will update some rows"               │
│   • S lock on table + IX for updates                                       │
│                                                                              │
│   Example:                                                                  │
│   UPDATE orders SET status = 'shipped' WHERE id = 123;                     │
│   1. Acquire IX on orders table                                            │
│   2. Acquire X on row 123                                                  │
│   3. Other transactions can still lock other rows                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Lock Granularity

### 3.1 Granularity Levels

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      LOCK GRANULARITY LEVELS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   DATABASE LOCK                                                             │
│   ├── SCHEMA LOCK                                                           │
│   │   ├── TABLE LOCK                                                        │
│   │   │   ├── PAGE/BLOCK LOCK                                              │
│   │   │   │   └── ROW LOCK                                                 │
│   │   │   └── KEY/INDEX LOCK                                               │
│   │   └── PARTITION LOCK                                                    │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │ Coarse Granularity    │  Fine Granularity                   │          │
│   │ (Database, Table)     │  (Row, Key)                         │          │
│   ├─────────────────────────────────────────────────────────────┤          │
│   │ ✓ Low overhead        │  ✗ High overhead                    │          │
│   │ ✗ Low concurrency     │  ✓ High concurrency                 │          │
│   │ ✓ Simple management   │  ✗ Complex management               │          │
│   │ ✓ Good for bulk ops   │  ✓ Good for OLTP                   │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Lock Escalation

```sql
-- Lock escalation: Automatically convert many row locks to table lock
-- Reduces overhead when too many row locks

-- SQL Server: Configure escalation threshold
ALTER TABLE orders SET (LOCK_ESCALATION = AUTO);   -- Default
ALTER TABLE orders SET (LOCK_ESCALATION = TABLE);  -- Always table
ALTER TABLE orders SET (LOCK_ESCALATION = DISABLE); -- Never escalate

-- When escalation happens:
-- 1. Transaction holds many row locks on a table
-- 2. Memory for lock tracking is high
-- 3. System converts to single table lock
-- 4. May reduce concurrency temporarily
```

---

## 4. Row-Level Locking

### 4.1 PostgreSQL Row Locking

```sql
-- FOR UPDATE: Exclusive lock for modification
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Other transactions block on SELECT FOR UPDATE or UPDATE

-- FOR NO KEY UPDATE: Weaker exclusive lock
SELECT * FROM accounts WHERE id = 1 FOR NO KEY UPDATE;
-- Allows FOR KEY SHARE, blocks FOR UPDATE

-- FOR SHARE: Shared lock
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
-- Allows other FOR SHARE, blocks FOR UPDATE

-- FOR KEY SHARE: Weakest lock
SELECT * FROM accounts WHERE id = 1 FOR KEY SHARE;
-- Only blocks FOR UPDATE, allows FOR NO KEY UPDATE

-- NOWAIT: Fail immediately if lock not available
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;

-- SKIP LOCKED: Skip locked rows
SELECT * FROM jobs WHERE status = 'pending'
FOR UPDATE SKIP LOCKED LIMIT 10;
-- Great for queue processing - gets unlocked rows only
```

### 4.2 MySQL/InnoDB Row Locking

```sql
-- FOR UPDATE: Exclusive lock
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- FOR SHARE (MySQL 8.0+) / LOCK IN SHARE MODE (older)
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;  -- Older syntax

-- NOWAIT and SKIP LOCKED (MySQL 8.0+)
SELECT * FROM jobs WHERE status = 'pending'
FOR UPDATE NOWAIT;

SELECT * FROM jobs WHERE status = 'pending'
FOR UPDATE SKIP LOCKED;
```

---

## 5. Index and Gap Locking

### 5.1 Record Locks

```sql
-- Lock on the index record (the row)
-- InnoDB example:
SELECT * FROM orders WHERE id = 100 FOR UPDATE;
-- Locks the index entry for id = 100
```

### 5.2 Gap Locks

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          GAP LOCKS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Gap lock: Lock on the "gap" between index values                         │
│   Prevents inserts into the gap (phantom prevention)                        │
│                                                                              │
│   Index values:    [10] ─── gap ─── [20] ─── gap ─── [30]                  │
│                                                                              │
│   SELECT * FROM t WHERE id BETWEEN 15 AND 25 FOR UPDATE;                   │
│                                                                              │
│   Locks:                                                                    │
│   • Gap before 20 (prevents insert of 15-19)                               │
│   • Record 20                                                              │
│   • Gap after 20 (prevents insert of 21-24)                                │
│                                                                              │
│   Another transaction:                                                      │
│   INSERT INTO t (id) VALUES (18);  -- BLOCKED                              │
│   INSERT INTO t (id) VALUES (5);   -- ALLOWED                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Next-Key Locks

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       NEXT-KEY LOCKS                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Next-Key Lock = Record Lock + Gap Lock before it                          │
│                                                                              │
│   InnoDB default locking in REPEATABLE READ                                 │
│                                                                              │
│   Index: 10, 20, 30                                                         │
│                                                                              │
│   SELECT * FROM t WHERE id = 20 FOR UPDATE;                                 │
│                                                                              │
│   Next-key locks on:                                                        │
│   • (10, 20] - gap from 10 to 20, plus record 20                           │
│                                                                              │
│   This prevents:                                                            │
│   • Updates to record 20                                                   │
│   • Inserts of values 11-20                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Table-Level Locks

### 6.1 Explicit Table Locks

```sql
-- PostgreSQL
LOCK TABLE accounts IN ACCESS SHARE MODE;           -- Weakest
LOCK TABLE accounts IN ROW SHARE MODE;
LOCK TABLE accounts IN ROW EXCLUSIVE MODE;
LOCK TABLE accounts IN SHARE UPDATE EXCLUSIVE MODE;
LOCK TABLE accounts IN SHARE MODE;
LOCK TABLE accounts IN SHARE ROW EXCLUSIVE MODE;
LOCK TABLE accounts IN EXCLUSIVE MODE;
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;       -- Strongest

-- MySQL
LOCK TABLES accounts READ;                          -- Shared
LOCK TABLES accounts WRITE;                         -- Exclusive
LOCK TABLES accounts READ, orders WRITE;            -- Multiple tables
UNLOCK TABLES;                                      -- Release all

-- SQL Server
-- Uses hints instead of explicit lock statements
SELECT * FROM accounts WITH (TABLOCK);              -- Table lock
SELECT * FROM accounts WITH (TABLOCKX);             -- Exclusive table lock
SELECT * FROM accounts WITH (HOLDLOCK);             -- Hold lock until commit
```

### 6.2 PostgreSQL Lock Mode Compatibility

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              POSTGRESQL TABLE LOCK COMPATIBILITY                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Requested Mode              Conflicts With                               │
│   ──────────────────────────────────────────────────────────────────────   │
│   ACCESS SHARE               ACCESS EXCLUSIVE                              │
│   ROW SHARE                  EXCLUSIVE, ACCESS EXCLUSIVE                   │
│   ROW EXCLUSIVE              SHARE, SHARE ROW EXCL, EXCL, ACCESS EXCL     │
│   SHARE UPDATE EXCL          SHARE UPDATE EXCL, SHARE, SHARE ROW, X, AX   │
│   SHARE                      ROW EXCL, SHARE UPDATE, SHARE ROW, X, AX     │
│   SHARE ROW EXCLUSIVE        ROW EXCL, SHARE UPDATE, SHARE, SRX, X, AX    │
│   EXCLUSIVE                  ROW SHARE, ROW EXCL, SU, S, SRX, X, AX       │
│   ACCESS EXCLUSIVE           ALL (blocks everything)                       │
│                                                                              │
│   Common operations and their locks:                                       │
│   • SELECT                     → ACCESS SHARE                              │
│   • SELECT FOR UPDATE          → ROW SHARE                                 │
│   • INSERT/UPDATE/DELETE       → ROW EXCLUSIVE                             │
│   • CREATE INDEX               → SHARE                                     │
│   • ALTER TABLE, DROP, VACUUM  → ACCESS EXCLUSIVE                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Advisory Locks

### 7.1 PostgreSQL Advisory Locks

```sql
-- Application-defined locks (not tied to database objects)

-- Session-level advisory lock (held until session ends or explicit unlock)
SELECT pg_advisory_lock(12345);              -- Blocking acquire
SELECT pg_try_advisory_lock(12345);          -- Non-blocking (returns boolean)
SELECT pg_advisory_unlock(12345);            -- Release

-- Transaction-level advisory lock (released at transaction end)
SELECT pg_advisory_xact_lock(12345);
SELECT pg_try_advisory_xact_lock(12345);
-- No unlock needed - released on COMMIT/ROLLBACK

-- Shared advisory locks
SELECT pg_advisory_lock_shared(12345);       -- Shared
SELECT pg_try_advisory_lock_shared(12345);
SELECT pg_advisory_unlock_shared(12345);

-- Two-key advisory locks (more namespace)
SELECT pg_advisory_lock(classid, objid);     -- Two integers
SELECT pg_advisory_lock(123, 456);           -- Lock (123, 456)

-- Check current advisory locks
SELECT * FROM pg_locks WHERE locktype = 'advisory';
```

### 7.2 Advisory Lock Use Cases

```python
# Python: Using advisory locks for coordination

import psycopg2

def process_with_lock(job_id):
    conn = psycopg2.connect("dbname=mydb")
    cur = conn.cursor()

    # Try to acquire lock
    cur.execute("SELECT pg_try_advisory_lock(%s)", (job_id,))
    got_lock = cur.fetchone()[0]

    if not got_lock:
        print(f"Job {job_id} already being processed")
        return

    try:
        # Process the job
        cur.execute("UPDATE jobs SET status = 'processing' WHERE id = %s", (job_id,))
        # ... do work ...
        cur.execute("UPDATE jobs SET status = 'complete' WHERE id = %s", (job_id,))
        conn.commit()
    finally:
        # Release lock
        cur.execute("SELECT pg_advisory_unlock(%s)", (job_id,))
        conn.close()


# Distributed mutex pattern
def distributed_singleton(name_hash):
    """Ensure only one instance runs across all processes."""
    conn = psycopg2.connect("dbname=mydb")
    cur = conn.cursor()

    # Block until we get the lock
    cur.execute("SELECT pg_advisory_lock(%s)", (name_hash,))
    return conn  # Caller must keep connection open
```

---

## 8. Lock Monitoring

### 8.1 PostgreSQL Lock Views

```sql
-- View all locks
SELECT
    locktype,
    relation::regclass,
    mode,
    granted,
    pid,
    pg_blocking_pids(pid) AS blocked_by
FROM pg_locks
WHERE relation IS NOT NULL;

-- View lock waits
SELECT
    blocked.pid AS blocked_pid,
    blocked.query AS blocked_query,
    blocking.pid AS blocking_pid,
    blocking.query AS blocking_query,
    blocking.state AS blocking_state
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
    ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE blocked.pid != blocking.pid;

-- Detailed lock info
SELECT
    l.locktype,
    l.relation::regclass AS table,
    l.page,
    l.tuple,
    l.mode,
    l.granted,
    a.usename,
    a.query,
    a.query_start,
    age(now(), a.query_start) AS query_age
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.relation IS NOT NULL
ORDER BY a.query_start;
```

### 8.2 MySQL Lock Monitoring

```sql
-- InnoDB lock waits
SELECT * FROM information_schema.innodb_lock_waits;

-- Current locks
SELECT * FROM performance_schema.data_locks;

-- Lock wait events
SELECT * FROM performance_schema.data_lock_waits;

-- Metadata locks
SELECT * FROM performance_schema.metadata_locks;

-- Show engine status (includes lock info)
SHOW ENGINE INNODB STATUS;
```

---

## 9. Lock Timeouts and Handling

### 9.1 Setting Timeouts

```sql
-- PostgreSQL
SET lock_timeout = '5s';                    -- Wait max 5 seconds for lock
SET deadlock_timeout = '1s';                -- Check for deadlock after 1s

-- MySQL
SET innodb_lock_wait_timeout = 50;          -- Wait max 50 seconds

-- SQL Server
SET LOCK_TIMEOUT 5000;                      -- Wait max 5000 ms
```

### 9.2 Handling Lock Failures

```python
# Python: Retry with backoff on lock timeout
import time
import psycopg2

def update_with_retry(account_id, amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            conn = psycopg2.connect("dbname=mydb")
            conn.set_session(autocommit=False)
            cur = conn.cursor()

            # Set lock timeout for this session
            cur.execute("SET lock_timeout = '2s'")

            cur.execute("""
                SELECT balance FROM accounts
                WHERE id = %s FOR UPDATE
            """, (account_id,))

            balance = cur.fetchone()[0]
            new_balance = balance + amount

            cur.execute("""
                UPDATE accounts SET balance = %s WHERE id = %s
            """, (new_balance, account_id))

            conn.commit()
            return True

        except psycopg2.errors.LockNotAvailable:
            print(f"Lock timeout, attempt {attempt + 1}")
            conn.rollback()
            time.sleep(0.5 * (2 ** attempt))  # Exponential backoff

        except psycopg2.errors.DeadlockDetected:
            print(f"Deadlock detected, attempt {attempt + 1}")
            conn.rollback()
            time.sleep(0.1)

        finally:
            conn.close()

    raise Exception("Failed after max retries")
```

---

## 10. Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LOCKING BEST PRACTICES                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. MINIMIZE LOCK DURATION                                                 │
│      • Keep transactions short                                             │
│      • Lock resources as late as possible                                  │
│      • Don't hold locks during external calls or user input                │
│                                                                              │
│   2. USE APPROPRIATE GRANULARITY                                            │
│      • Row locks for OLTP workloads                                        │
│      • Table locks for batch operations                                    │
│      • Let the database choose when possible                               │
│                                                                              │
│   3. ACCESS RESOURCES IN CONSISTENT ORDER                                   │
│      • Prevents deadlocks                                                  │
│      • Document the ordering convention                                    │
│      • Apply to all code paths                                             │
│                                                                              │
│   4. HANDLE FAILURES GRACEFULLY                                             │
│      • Set reasonable timeouts                                             │
│      • Implement retry logic                                               │
│      • Log and monitor lock issues                                         │
│                                                                              │
│   5. CONSIDER OPTIMISTIC LOCKING                                            │
│      • For read-heavy workloads                                           │
│      • When conflicts are rare                                             │
│      • Using version columns                                               │
│                                                                              │
│   6. MONITOR LOCK ACTIVITY                                                  │
│      • Track lock wait times                                               │
│      • Alert on deadlocks                                                  │
│      • Review slow queries for lock issues                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. Summary

| Lock Type | Purpose | Compatibility |
|-----------|---------|---------------|
| Shared (S) | Read access | Multiple holders allowed |
| Exclusive (X) | Write access | Single holder only |
| Intent (IS/IX) | Signal sub-level locking | Compatible with other intents |
| Gap | Prevent inserts in range | Blocks inserts only |
| Advisory | Application coordination | Custom compatibility |

**Key Points:**
- Use row-level locking for maximum concurrency
- Intent locks enable efficient multi-granularity locking
- Gap locks prevent phantom reads in REPEATABLE READ
- Advisory locks are great for application-level coordination
- Always handle lock timeouts and deadlocks
- Monitor locks to identify performance issues
