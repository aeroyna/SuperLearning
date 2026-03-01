# Isolation Levels

## 1. Introduction

**Isolation levels** define the degree to which transactions are isolated from each other's modifications. Higher isolation provides more consistency guarantees but typically reduces concurrency.

```mermaid
flowchart LR
    subgraph Spectrum["Isolation Level Spectrum"]
        direction LR
        RU["READ<br/>UNCOMMITTED<br/><br/>\"Anything goes\""]
        RC["READ<br/>COMMITTED<br/><br/>\"See only<br/>committed data\""]
        RR["REPEATABLE<br/>READ<br/><br/>\"Same data<br/>if reread\""]
        S["SERIALIZABLE<br/><br/>\"As if running<br/>one at a time\""]
    end

    RU -->|"More Isolation →"| RC
    RC -->|"More Isolation →"| RR
    RR -->|"More Isolation →"| S

    Less["⚡ Less Isolation<br/>✅ More Concurrency<br/>⚠️ More Anomalies"] -.-> RU
    More["🔒 More Isolation<br/>⏳ Less Concurrency<br/>✅ Fewer Anomalies"] -.-> S

    style RU fill:#ffcccc
    style RC fill:#ffe6cc
    style RR fill:#ffffcc
    style S fill:#ccffcc
```

---

## 2. Read Phenomena (Anomalies)

### 2.1 Dirty Read

Reading uncommitted changes from another transaction.

```mermaid
sequenceDiagram
    participant A as Transaction A
    participant DB as Database
    participant B as Transaction B

    Note over A,B: 💀 DIRTY READ ANOMALY

    A->>DB: BEGIN
    A->>DB: UPDATE balance = 500<br/>(was 1000)
    Note over DB: Balance: 500<br/>(uncommitted!)

    B->>DB: BEGIN
    B->>DB: SELECT balance WHERE id=1
    DB-->>B: Returns 500 ⚠️

    A->>DB: ROLLBACK ❌
    Note over DB: Balance back to 1000

    Note over B: ❌ Used 500 for calculations<br/>but actual value is 1000!
    Note over A,B: Problem: B read data that never actually existed
```

### 2.2 Non-Repeatable Read

Same query returns different results within one transaction.

```mermaid
sequenceDiagram
    participant A as Transaction A
    participant DB as Database
    participant B as Transaction B

    Note over A,B: 🔄 NON-REPEATABLE READ ANOMALY

    A->>DB: BEGIN
    A->>DB: SELECT balance WHERE id=1
    DB-->>A: Returns 1000 ✅

    B->>DB: BEGIN
    B->>DB: UPDATE balance = 500
    B->>DB: COMMIT ✅
    Note over DB: Balance now: 500

    A->>DB: SELECT balance WHERE id=1
    DB-->>A: Returns 500 ⚠️

    Note over A: ❌ Same query returned<br/>different results!
    Note over A,B: Problem: Same query, same transaction, different results
```

### 2.3 Phantom Read

New rows appear that match a previous query's criteria.

```mermaid
sequenceDiagram
    participant A as Transaction A
    participant DB as Database
    participant B as Transaction B

    Note over A,B: 👻 PHANTOM READ ANOMALY

    A->>DB: BEGIN
    A->>DB: SELECT COUNT(*) WHERE status='pending'
    DB-->>A: Returns 5 rows ✅

    B->>DB: BEGIN
    B->>DB: INSERT INTO orders (status='pending')
    B->>DB: COMMIT ✅
    Note over DB: Now 6 pending orders

    A->>DB: SELECT COUNT(*) WHERE status='pending'
    DB-->>A: Returns 6 rows 👻

    Note over A: ❌ A new "phantom"<br/>row appeared!
    Note over A,B: Problem: Range query results changed by another transaction
```

### 2.4 Lost Update

Two transactions update the same row, one overwrites the other.

```mermaid
sequenceDiagram
    participant A as Transaction A
    participant DB as Database
    participant B as Transaction B

    Note over A,B: 💸 LOST UPDATE ANOMALY

    A->>DB: BEGIN
    B->>DB: BEGIN

    A->>DB: SELECT balance WHERE id=1
    DB-->>A: Returns 1000
    B->>DB: SELECT balance WHERE id=1
    DB-->>B: Returns 1000

    Note over A: Calculate: 1000 + 100 = 1100
    Note over B: Calculate: 1000 - 50 = 950

    A->>DB: UPDATE balance = 1100
    A->>DB: COMMIT ✅
    Note over DB: Balance: 1100

    B->>DB: UPDATE balance = 950
    B->>DB: COMMIT ✅
    Note over DB: Balance: 950 ⚠️

    Note over A,B: ❌ Final balance: 950 (should be 1050!)<br/>A's +100 update was LOST!
```

---

## 3. Isolation Levels Explained

### 3.1 READ UNCOMMITTED

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Allows: Dirty reads, Non-repeatable reads, Phantom reads
-- Prevents: Nothing

-- Use case: Very rare. Only for read-only analytics where
-- approximate results are acceptable and speed is critical
```

```
Phenomena Allowed:
✗ Dirty Read
✗ Non-Repeatable Read
✗ Phantom Read

Performance: Highest (no locking overhead)
Concurrency: Highest
Use Cases:
• Approximate counts/aggregations
• Monitoring dashboards (non-critical)
• Debug queries during development
```

### 3.2 READ COMMITTED

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Default for: PostgreSQL, Oracle, SQL Server
-- Allows: Non-repeatable reads, Phantom reads
-- Prevents: Dirty reads

-- Each statement sees a snapshot of committed data at statement start
```

```
Phenomena Allowed:
✓ Dirty Read (prevented)
✗ Non-Repeatable Read
✗ Phantom Read

Performance: Good
Concurrency: Good
Use Cases:
• Most OLTP applications
• Default for many databases
• Good balance of consistency and performance
```

```sql
-- Behavior example
BEGIN;

SELECT balance FROM accounts WHERE id = 1;  -- Snapshot at this moment
-- Another transaction commits a change
SELECT balance FROM accounts WHERE id = 1;  -- New snapshot, might differ

COMMIT;
```

### 3.3 REPEATABLE READ

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Default for: MySQL/InnoDB
-- Allows: Phantom reads (in standard SQL; PostgreSQL prevents them)
-- Prevents: Dirty reads, Non-repeatable reads

-- Transaction sees a snapshot from the start; re-reads return same data
```

```
Phenomena Allowed:
✓ Dirty Read (prevented)
✓ Non-Repeatable Read (prevented)
✗ Phantom Read (allowed in SQL standard, prevented in PostgreSQL)

Performance: Moderate
Concurrency: Moderate
Use Cases:
• Financial reports that read same data multiple times
• Long-running read transactions needing consistency
• Batch jobs processing data
```

```sql
-- Behavior example
BEGIN;

SELECT balance FROM accounts WHERE id = 1;  -- Snapshot for entire transaction
-- Another transaction commits a change
SELECT balance FROM accounts WHERE id = 1;  -- Same result as first query

COMMIT;
```

### 3.4 SERIALIZABLE

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Strictest level
-- Prevents: All phenomena including phantom reads
-- Behavior: As if transactions ran one at a time (serially)
```

```
Phenomena Allowed:
✓ Dirty Read (prevented)
✓ Non-Repeatable Read (prevented)
✓ Phantom Read (prevented)

Performance: Lowest
Concurrency: Lowest
Use Cases:
• Critical financial transactions
• When absolute correctness is required
• When you'd otherwise need application-level locking
```

---

## 4. Comparison Matrix

```mermaid
flowchart TB
    subgraph Matrix["Isolation Levels vs Anomalies"]
        direction TB
        subgraph RU["READ UNCOMMITTED"]
            RU1["❌ Dirty Read"]
            RU2["❌ Non-Repeatable"]
            RU3["❌ Phantom"]
            RU4["❌ Lost Update"]
        end
        subgraph RC["READ COMMITTED"]
            RC1["✅ Dirty Read"]
            RC2["❌ Non-Repeatable"]
            RC3["❌ Phantom"]
            RC4["❌ Lost Update"]
        end
        subgraph RR["REPEATABLE READ"]
            RR1["✅ Dirty Read"]
            RR2["✅ Non-Repeatable"]
            RR3["⚠️ Phantom*"]
            RR4["✅ Lost Update"]
        end
        subgraph SER["SERIALIZABLE"]
            SER1["✅ Dirty Read"]
            SER2["✅ Non-Repeatable"]
            SER3["✅ Phantom"]
            SER4["✅ Lost Update"]
        end
    end

    Note["* PostgreSQL RR prevents phantoms<br/>MySQL/InnoDB RR allows in some cases"]

    style RU fill:#ffcccc
    style RC fill:#ffe6cc
    style RR fill:#ffffcc
    style SER fill:#ccffcc
```

---

## 5. Implementation Approaches

### 5.1 Locking (Pessimistic)

```sql
-- Two-Phase Locking (2PL)
-- Transactions acquire locks, hold until commit/rollback

-- Shared lock (read lock) - multiple readers allowed
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- Exclusive lock (write lock) - one writer, no readers
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Lock types:
-- • Row locks: Lock specific rows
-- • Table locks: Lock entire table
-- • Gap locks: Lock gaps between index values (for phantom prevention)
```

### 5.2 MVCC (Optimistic)

```sql
-- Multi-Version Concurrency Control
-- Readers don't block writers; writers don't block readers

-- How it works:
-- 1. Each row has version info (transaction ID, timestamps)
-- 2. Writes create new versions instead of modifying in place
-- 3. Reads see consistent snapshot based on transaction start time
-- 4. Old versions cleaned up by vacuum/garbage collection

-- Used by: PostgreSQL, MySQL/InnoDB, Oracle
```

```mermaid
sequenceDiagram
    participant T1 as TXN 1 (Reader)
    participant DB as Database
    participant T2 as TXN 2 (Writer)

    Note over T1,T2: 📸 MVCC (Multi-Version Concurrency Control)
    Note over DB: v1: balance=100

    T1->>DB: BEGIN (snapshot at T1)
    T2->>DB: BEGIN

    T1->>DB: SELECT balance
    DB-->>T1: Returns 100 (v1) ✅

    T2->>DB: UPDATE balance = 200
    Note over DB: v1: 100 (old)<br/>v2: 200 (new, uncommitted)

    T1->>DB: SELECT balance
    DB-->>T1: Still returns 100 (v1) ✅
    Note over T1: Sees consistent snapshot!

    T2->>DB: COMMIT ✅
    Note over DB: v1: 100 (old)<br/>v2: 200 (committed)

    T1->>DB: SELECT balance
    DB-->>T1: STILL returns 100 (v1) ✅
    Note over T1: Snapshot isolation!

    T1->>DB: COMMIT
    Note over DB: v1: garbage collected<br/>v2: 200

    Note over T1,T2: ✅ TXN 1 sees consistent snapshot<br/>even as TXN 2 commits!
```

---

## 6. Choosing an Isolation Level

```mermaid
flowchart TD
    Start{{"🤔 Which Isolation Level?"}}

    Start --> Q1{"High concurrency<br/>+ short transactions?"}
    Q1 -->|Yes| RC["✅ READ COMMITTED<br/><br/>Default OLTP workloads<br/>High concurrency<br/>Short transactions<br/>Occasional inconsistency OK"]

    Q1 -->|No| Q2{"Need consistent<br/>re-reads?"}
    Q2 -->|Yes| RR["✅ REPEATABLE READ<br/><br/>Multiple reads need consistency<br/>Report generation<br/>Batch processing<br/>Longer transactions"]

    Q2 -->|No| Q3{"Correctness<br/>critical?"}
    Q3 -->|Yes| SER["✅ SERIALIZABLE<br/><br/>Financial transactions<br/>Correctness is critical<br/>Read-then-write patterns<br/>Can tolerate lower throughput"]

    Q3 -->|No| Q4{"Just monitoring<br/>or debugging?"}
    Q4 -->|Yes| RU["⚠️ READ UNCOMMITTED<br/><br/>Non-critical monitoring<br/>Approximate statistics<br/>Debugging only"]
    Q4 -->|No| RC

    style RC fill:#ffe6cc
    style RR fill:#ffffcc
    style SER fill:#ccffcc
    style RU fill:#ffcccc
```

---

## 7. Code Examples

### PostgreSQL

```sql
-- Set for session
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Set for specific transaction
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
    SELECT * FROM accounts WHERE id = 1;
    -- more operations...
COMMIT;

-- Check current level
SHOW transaction_isolation;
```

### Python (psycopg2)

```python
import psycopg2
from psycopg2 import extensions

conn = psycopg2.connect(database="mydb")

# Set isolation level
conn.set_isolation_level(extensions.ISOLATION_LEVEL_SERIALIZABLE)

# Or per transaction
conn.set_isolation_level(extensions.ISOLATION_LEVEL_REPEATABLE_READ)

try:
    with conn.cursor() as cur:
        cur.execute("SELECT balance FROM accounts WHERE id = %s", (1,))
        balance = cur.fetchone()[0]
        # ... calculations ...
        cur.execute("UPDATE accounts SET balance = %s WHERE id = %s",
                   (new_balance, 1))
    conn.commit()
except psycopg2.errors.SerializationFailure:
    conn.rollback()
    # Retry the transaction
```

### Java (JDBC)

```java
Connection conn = dataSource.getConnection();

// Set isolation level
conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);

// Available constants:
// Connection.TRANSACTION_READ_UNCOMMITTED
// Connection.TRANSACTION_READ_COMMITTED
// Connection.TRANSACTION_REPEATABLE_READ
// Connection.TRANSACTION_SERIALIZABLE

try {
    conn.setAutoCommit(false);
    // Transaction operations...
    conn.commit();
} catch (SQLException e) {
    conn.rollback();
    if (e.getSQLState().equals("40001")) {  // Serialization failure
        // Retry transaction
    }
}
```

---

## 8. Summary

| Level | Dirty Read | Non-Repeatable | Phantom | Use Case |
|-------|------------|----------------|---------|----------|
| READ UNCOMMITTED | Possible | Possible | Possible | Rare, monitoring |
| READ COMMITTED | Prevented | Possible | Possible | Default OLTP |
| REPEATABLE READ | Prevented | Prevented | Possible* | Reports, batches |
| SERIALIZABLE | Prevented | Prevented | Prevented | Critical finance |

**Key Points:**
- Default to READ COMMITTED for most applications
- Use REPEATABLE READ for consistent multi-read transactions
- Use SERIALIZABLE for critical correctness requirements
- Be prepared to retry transactions at higher isolation levels
- Understand your database's specific implementation (MVCC vs locking)
