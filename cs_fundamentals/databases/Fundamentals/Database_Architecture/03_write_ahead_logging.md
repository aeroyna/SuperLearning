# Write-Ahead Logging (WAL)

## 1. Introduction

**Write-Ahead Logging (WAL)** is a technique that ensures data integrity by recording changes to a log before applying them to the actual data files. It's fundamental to ACID compliance and crash recovery.

```mermaid
flowchart TB
    subgraph Rule["📜 THE WAL RULE"]
        R1["Never write data pages to disk BEFORE<br/>log records have been flushed to stable storage"]
    end

    subgraph Process["WAL Process"]
        TXN["UPDATE accounts SET balance = 500<br/>WHERE id = 1"]
        S1["1️⃣ Write log record to WAL buffer"]
        S2["2️⃣ On COMMIT: Flush WAL to disk"]
        S3["3️⃣ Data page can stay dirty in memory"]
        S4["4️⃣ Background process writes data pages later"]

        TXN --> S1 --> S2 --> S3 --> S4
    end

    subgraph Why["❓ Why This Works"]
        W1["⚡ Sequential writes to WAL (fast)"]
        W2["🐢 Random writes to data pages (slow)"]
        W3["🔄 Crash? Replay WAL to recover"]
    end

    style Rule fill:#ffffcc
    style S2 fill:#ccffcc
```

---

## 2. WAL Components

### 2.1 Log Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       WAL LOG STRUCTURE                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   WAL FILE (Segment):                                                       │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │ Record │ Record │ Record │ Record │ Record │ Record │ ...       │      │
│   │   1    │   2    │   3    │   4    │   5    │   6    │           │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                                                                              │
│   LOG RECORD:                                                               │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │ LSN │ Prev │ TxID │ Type │ Page │ Offset │ Before │ After │ CRC │      │
│   │     │ LSN  │      │      │ ID   │        │ Image  │ Image │     │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                                                                              │
│   Fields:                                                                   │
│   • LSN (Log Sequence Number): Unique identifier, monotonically increasing│
│   • Prev LSN: Previous record in same transaction                         │
│   • TxID: Transaction identifier                                          │
│   • Type: INSERT, UPDATE, DELETE, COMMIT, ABORT, CHECKPOINT               │
│   • Page ID: Which page was modified                                      │
│   • Before Image: Old value (for UNDO)                                    │
│   • After Image: New value (for REDO)                                     │
│   • CRC: Checksum for integrity                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 LSN (Log Sequence Number)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LOG SEQUENCE NUMBER (LSN)                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   LSN uniquely identifies each log record                                  │
│   Used to track recovery position and page freshness                       │
│                                                                              │
│   Components (PostgreSQL):                                                  │
│   • Segment number (file)                                                  │
│   • Offset within segment                                                  │
│   • Example: 0/16B3A40 (segment 0, offset 0x16B3A40)                       │
│                                                                              │
│   LSN stored in:                                                            │
│   • Each log record (its own LSN)                                         │
│   • Each data page (PageLSN = last WAL record that modified it)           │
│   • Checkpoint record (recovery start point)                              │
│                                                                              │
│   During recovery:                                                          │
│   ┌──────────────────────────────────────────────────────────────────┐     │
│   │ For each log record with LSN X:                                  │     │
│   │   If page's PageLSN < X:                                        │     │
│   │     → Apply this log record (REDO)                              │     │
│   │   Else:                                                          │     │
│   │     → Skip (page already has this change)                       │     │
│   └──────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. WAL Write Process

```mermaid
sequenceDiagram
    participant App as Application
    participant WALBuf as WAL Buffer (RAM)
    participant BufPool as Buffer Pool
    participant WALDisk as WAL on Disk
    participant DataDisk as Data Pages on Disk

    Note over App,DataDisk: 📝 WAL Write Process

    App->>App: 1️⃣ UPDATE balance = 500

    App->>WALBuf: 2️⃣ Write log record
    Note over WALBuf: [Rec1][Rec2][NewRec]

    App->>BufPool: 3️⃣ Modify data page
    Note over BufPool: Page marked DIRTY<br/>PageLSN updated

    App->>App: 4️⃣ COMMIT requested

    WALBuf->>WALDisk: 5️⃣ Flush WAL (fsync)
    Note over WALDisk: [...records...][NewRec]

    WALDisk-->>App: 6️⃣ SUCCESS ✅
    Note over App: Transaction is DURABLE

    Note over BufPool,DataDisk: 7️⃣ Background: Data page written later
    BufPool-)DataDisk: Checkpoint / Background writer
```

---

## 4. Recovery Process

### 4.1 ARIES Recovery Algorithm

```mermaid
flowchart TB
    Title["🔄 ARIES Recovery Algorithm<br/>(Algorithms for Recovery and Isolation Exploiting Semantics)"]

    subgraph Phase1["📊 PHASE 1: ANALYSIS"]
        A1["Scan log from last checkpoint"]
        A2["Build dirty page table"]
        A3["Build active transaction table"]
        A4["Determine REDO start point"]
        A1 --> A2 --> A3 --> A4
    end

    subgraph Phase2["▶️ PHASE 2: REDO"]
        R1["Replay log FORWARD from REDO point"]
        R2["Redo ALL operations (committed + uncommitted)"]
        R3["Brings DB to crash-time state"]
        R4["Skip if PageLSN >= record LSN"]
        R1 --> R2 --> R3 --> R4
    end

    subgraph Phase3["◀️ PHASE 3: UNDO"]
        U1["For each uncommitted transaction"]
        U2["Follow Prev LSN chain BACKWARD"]
        U3["Undo each operation"]
        U4["Write compensation log records (CLR)"]
        U1 --> U2 --> U3 --> U4
    end

    Title --> Phase1
    Phase1 --> Phase2
    Phase2 --> Phase3

    style Phase1 fill:#e6f3ff
    style Phase2 fill:#e6ffe6
    style Phase3 fill:#ffe6e6
```

### 4.2 Recovery Example

```
Log before crash:
┌─────────────────────────────────────────────────────────────────────────────┐
│ LSN │ TxID │ Type   │ Details                                               │
├─────┼──────┼────────┼───────────────────────────────────────────────────────┤
│ 10  │  -   │ CKPT   │ Checkpoint                                            │
│ 11  │ T1   │ UPDATE │ Page 5: balance 1000 → 900                           │
│ 12  │ T2   │ UPDATE │ Page 3: balance 500 → 600                            │
│ 13  │ T1   │ UPDATE │ Page 7: balance 200 → 100                            │
│ 14  │ T1   │ COMMIT │                                                       │
│ 15  │ T2   │ UPDATE │ Page 3: balance 600 → 700                            │
│     │      │ CRASH! │                                                       │
└─────────────────────────────────────────────────────────────────────────────┘

Recovery:
ANALYSIS: T1 committed, T2 active (uncommitted)
REDO: Apply LSN 11, 12, 13, 14, 15 (if needed based on PageLSN)
UNDO: Rollback T2 - undo LSN 15, then LSN 12
```

---

## 5. Checkpoints

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CHECKPOINTS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A checkpoint is a known-good recovery point                              │
│                                                                              │
│   PURPOSE:                                                                  │
│   • Limit recovery time (don't replay entire log)                         │
│   • Allow WAL file recycling                                               │
│   • Flush dirty pages to reduce crash recovery I/O                        │
│                                                                              │
│   CHECKPOINT PROCESS:                                                       │
│   1. Write checkpoint start record to WAL                                  │
│   2. Flush all dirty pages to disk                                        │
│   3. Write checkpoint complete record to WAL                              │
│   4. Update checkpoint location in control file                           │
│   5. Old WAL files can be recycled                                        │
│                                                                              │
│   TYPES:                                                                    │
│   • Sharp Checkpoint: All dirty pages flushed (stops processing)          │
│   • Fuzzy Checkpoint: Pages flushed gradually (PostgreSQL default)        │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │        WAL Timeline                                              │      │
│   │                                                                   │      │
│   │ [records...][CKPT][records...][CKPT][records...][CRASH]         │      │
│   │              ▲                  ▲                                │      │
│   │              │                  │                                │      │
│   │         Old checkpoint     Last checkpoint                       │      │
│   │         (can recycle)      (recovery starts here)               │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. WAL Configuration

### 6.1 PostgreSQL

```sql
-- WAL directory
-- $PGDATA/pg_wal/

-- Check WAL location
SHOW data_directory;
SELECT pg_current_wal_lsn();

-- WAL settings
SHOW wal_level;              -- minimal, replica, logical
SHOW wal_buffers;            -- WAL buffer size (default: 1/32 shared_buffers)
SHOW checkpoint_timeout;     -- Time between checkpoints (default: 5min)
SHOW max_wal_size;           -- Max WAL size before checkpoint (default: 1GB)
SHOW min_wal_size;           -- Min WAL to keep (default: 80MB)

-- Synchronous commit options
SHOW synchronous_commit;
-- on: Wait for WAL flush (default, safest)
-- off: Return before flush (faster, risk of data loss)
-- remote_write: Wait for replica to receive
-- remote_apply: Wait for replica to apply

-- For high-throughput writes
SET synchronous_commit = off;  -- Per-session, not recommended for critical data

-- WAL compression (PostgreSQL 15+)
SHOW wal_compression;  -- off, pglz, lz4, zstd
```

### 6.2 MySQL/InnoDB

```sql
-- Redo log files
-- /var/lib/mysql/ib_logfile0, ib_logfile1

-- Check redo log settings
SHOW VARIABLES LIKE 'innodb_log%';

-- Key settings:
-- innodb_log_file_size: Size of each log file (default: 50MB, recommend: 1-2GB)
-- innodb_log_files_in_group: Number of log files (default: 2)
-- innodb_log_buffer_size: Buffer before writing (default: 16MB)

-- Flush behavior
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
-- 1: Flush on every commit (safest, default)
-- 0: Flush every second (fastest, risk of 1s data loss)
-- 2: Write to OS buffer on commit, flush every second

-- Flush method
SHOW VARIABLES LIKE 'innodb_flush_method';
-- O_DIRECT: Bypass OS cache (recommended for dedicated server)
-- fsync: Default on some systems
```

---

## 7. WAL for Replication

```mermaid
flowchart LR
    subgraph Physical["🔧 PHYSICAL REPLICATION"]
        P1["👑 PRIMARY<br/>[Write WAL]"]
        P2["📋 REPLICA<br/>[Apply WAL]"]
        P1 -->|"WAL Stream<br/>(byte-for-byte)"| P2
    end

    subgraph PhysicalNotes["Physical Features"]
        PN1["✅ Exact same WAL records"]
        PN2["✅ Byte-identical database"]
        PN3["❌ Can't cross versions"]
    end

    subgraph Logical["📝 LOGICAL REPLICATION"]
        L1["👑 PRIMARY<br/>[Write WAL]"]
        L2["📋 REPLICA<br/>[Apply SQL]"]
        L1 -->|"Decoded Changes<br/>(INSERT, UPDATE)"| L2
    end

    subgraph LogicalNotes["Logical Features"]
        LN1["✅ Cross-version replication"]
        LN2["✅ Selective table replication"]
        LN3["✅ Change data capture (CDC)"]
        LN4["📦 PostgreSQL: wal_level = logical<br/>MySQL: Binary Log (binlog)"]
    end

    style Physical fill:#e6f3ff
    style Logical fill:#e6ffe6
```

---

## 8. WAL Performance Considerations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 WAL PERFORMANCE OPTIMIZATION                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   BOTTLENECKS:                                                              │
│   • WAL flush on every commit (synchronous_commit = on)                   │
│   • Small transactions = many flushes                                     │
│   • WAL on slow disk                                                      │
│                                                                              │
│   SOLUTIONS:                                                                │
│                                                                              │
│   1. GROUP COMMIT                                                          │
│      • Multiple transactions share single WAL flush                       │
│      • Automatic in PostgreSQL when load is high                         │
│      • MySQL: binlog_group_commit_sync_delay                             │
│                                                                              │
│   2. SEPARATE WAL DISK                                                     │
│      • Dedicated fast storage for WAL                                     │
│      • SSD or NVMe for WAL, HDD for data                                 │
│      • Reduces contention                                                 │
│                                                                              │
│   3. BATCH TRANSACTIONS                                                    │
│      • Combine multiple operations in single transaction                  │
│      • One flush per transaction instead of per statement                │
│                                                                              │
│   4. ASYNC COMMIT (careful!)                                               │
│      • PostgreSQL: synchronous_commit = off                              │
│      • MySQL: innodb_flush_log_at_trx_commit = 0                         │
│      • Risk: Up to 3 * wal_writer_delay data loss                        │
│                                                                              │
│   5. WAL COMPRESSION                                                       │
│      • Reduces I/O at cost of CPU                                        │
│      • Good for WAL shipping/replication                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

| Concept | Purpose | Key Point |
|---------|---------|-----------|
| WAL Rule | Data integrity | Log before data page write |
| LSN | Ordering | Unique, monotonically increasing |
| Checkpoint | Recovery limit | Truncation point for WAL |
| REDO | Forward recovery | Replay committed operations |
| UNDO | Backward recovery | Rollback uncommitted transactions |

**WAL Benefits:**
- Crash recovery without data loss
- Enables point-in-time recovery
- Foundation for replication
- Allows lazy data page writes

**Key Settings:**
- PostgreSQL: `wal_level`, `synchronous_commit`, `checkpoint_timeout`
- MySQL: `innodb_flush_log_at_trx_commit`, `innodb_log_file_size`
