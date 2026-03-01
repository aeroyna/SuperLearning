# Binary Log Replication

## Learning Objectives
- Understand binary log structure and formats
- Learn replication event flow
- Master GTID-based replication
- Configure binary logging properly

---

## 1. Binary Log Fundamentals

### What is the Binary Log?

The binary log (binlog) records all changes to the database in a format that can be replayed.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Binary Log Structure                              │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                Binary Log Files                              │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │    │
│  │  │ binlog.000001│  │ binlog.000002│  │ binlog.000003│       │    │
│  │  │ (completed)  │  │ (completed)  │  │ (current)    │       │    │
│  │  └──────────────┘  └──────────────┘  └──────────────┘       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                Events in Binary Log                          │    │
│  │  ┌─────────────────────────────────────────────────────┐    │    │
│  │  │ [Header] [GTID Event] [Query Event]                 │    │    │
│  │  │ [Header] [GTID Event] [Table Map] [Row Event]       │    │    │
│  │  │ [Header] [GTID Event] [Query Event]                 │    │    │
│  │  └─────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Binary Log Configuration

```sql
-- View binary log settings
SHOW VARIABLES LIKE 'log_bin%';
SHOW VARIABLES LIKE 'binlog%';

-- my.cnf configuration
[mysqld]
# Enable binary logging
log_bin = /var/lib/mysql/binlog
server_id = 1                     # Unique server ID

# Binary log format
binlog_format = ROW               # STATEMENT, ROW, or MIXED

# Retention
binlog_expire_logs_seconds = 604800  # 7 days (MySQL 8.0)
# expire_logs_days = 7             # Legacy (pre-8.0)

# Size limit per file
max_binlog_size = 1G

# Durability
sync_binlog = 1                   # Sync on every commit (safest)
```

---

## 2. Binary Log Formats

### Statement-Based Replication (SBR)

```sql
-- Logs the SQL statement
binlog_format = STATEMENT

-- Advantages:
-- - Smaller log size
-- - Logs full SQL context

-- Disadvantages:
-- - Non-deterministic functions (NOW(), RAND()) may differ
-- - UDFs may not replicate correctly
-- - Some statements may produce warnings

-- Example log entry:
-- INSERT INTO orders (user_id, total) VALUES (123, 99.99)
```

### Row-Based Replication (RBR)

```sql
-- Logs actual row changes
binlog_format = ROW

-- Advantages:
-- - Deterministic replication
-- - Works with all statements
-- - Safer for complex queries

-- Disadvantages:
-- - Larger log size
-- - Less human-readable

-- Example: Logs the actual row data before/after change
-- Not the SQL statement
```

### Mixed Format

```sql
-- Chooses format per statement
binlog_format = MIXED

-- Uses STATEMENT by default
-- Switches to ROW for:
-- - Non-deterministic functions
-- - Statements with AUTO_INCREMENT
-- - UDF calls
```

### Row Image Settings

```sql
-- Control what's logged in ROW format
SHOW VARIABLES LIKE 'binlog_row_image';

-- Options:
-- FULL: All columns (default)
-- MINIMAL: Only changed columns + key columns
-- NOBLOB: All except BLOB/TEXT unchanged columns

SET GLOBAL binlog_row_image = 'MINIMAL';  -- Reduces log size
```

---

## 3. Viewing Binary Logs

### List Binary Logs

```sql
-- Show all binary log files
SHOW BINARY LOGS;
SHOW MASTER LOGS;

-- Current log position
SHOW MASTER STATUS;

+---------------+----------+--------------+------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+---------------+----------+--------------+------------------+
| binlog.000003 |      897 |              |                  |
+---------------+----------+--------------+------------------+
```

### Read Binary Log Events

```sql
-- Show events in current log
SHOW BINLOG EVENTS;

-- Show events in specific log
SHOW BINLOG EVENTS IN 'binlog.000003';

-- With offset and limit
SHOW BINLOG EVENTS IN 'binlog.000003' FROM 156 LIMIT 10;
```

### mysqlbinlog Utility

```bash
# View binary log contents
mysqlbinlog /var/lib/mysql/binlog.000003

# With readable output
mysqlbinlog --verbose /var/lib/mysql/binlog.000003

# Specific time range
mysqlbinlog --start-datetime="2024-01-15 10:00:00" \
            --stop-datetime="2024-01-15 12:00:00" \
            /var/lib/mysql/binlog.000003

# Specific position range
mysqlbinlog --start-position=156 --stop-position=897 \
            /var/lib/mysql/binlog.000003

# For specific database
mysqlbinlog --database=mydb /var/lib/mysql/binlog.000003

# Remote server
mysqlbinlog --read-from-remote-server \
            --host=master.example.com \
            --user=repl \
            --password \
            binlog.000003
```

---

## 4. GTID-Based Replication

### What is GTID?

Global Transaction Identifier uniquely identifies each transaction.

```
GTID Format: source_uuid:transaction_id

Example: 3E11FA47-71CA-11E1-9E33-C80AA9429562:23

Components:
- source_uuid: Server that originated the transaction
- transaction_id: Incrementing number per server
```

### Enable GTID

```sql
-- my.cnf configuration
[mysqld]
gtid_mode = ON
enforce_gtid_consistency = ON
log_bin = binlog
log_slave_updates = ON

-- Verify
SHOW VARIABLES LIKE 'gtid%';
```

### GTID Benefits

```sql
-- 1. Easy replica provisioning
-- No need to find binary log position

-- 2. Automatic position tracking
SHOW MASTER STATUS;
-- Shows: Executed_Gtid_Set

-- 3. Failover without log position coordination
-- Just point replica to new source

-- 4. No duplicate transactions
-- Server knows which GTIDs are already executed
```

### GTID Sets

```sql
-- View executed GTIDs
SELECT @@gtid_executed;

-- View purged GTIDs
SELECT @@gtid_purged;

-- Set GTID to skip a transaction
SET GTID_NEXT = '3E11FA47-71CA-11E1-9E33-C80AA9429562:23';
BEGIN;
COMMIT;  -- Empty transaction marks GTID as executed
SET GTID_NEXT = 'AUTOMATIC';
```

---

## 5. Replication Event Flow

### Source Server Processing

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Source (Master) Event Flow                         │
│                                                                      │
│  Client Transaction                                                  │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────┐                                                    │
│  │ InnoDB Engine│ ──────────────────────────────────┐                │
│  │  (Execute)   │                                   │                │
│  └──────────────┘                                   │                │
│       │                                             │                │
│       ▼                                             ▼                │
│  ┌──────────────┐                          ┌──────────────┐          │
│  │  Redo Log    │                          │  Binary Log  │          │
│  │  (Recovery)  │                          │ (Replication)│          │
│  └──────────────┘                          └──────────────┘          │
│                                                    │                 │
│                                                    ▼                 │
│                                            ┌──────────────┐          │
│                                            │ Binlog Dump  │          │
│                                            │   Thread     │          │
│                                            └──────┬───────┘          │
│                                                   │                  │
│                                                   ▼                  │
│                                            To Replicas               │
└─────────────────────────────────────────────────────────────────────┘
```

### Replica Server Processing

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Replica (Slave) Event Flow                         │
│                                                                      │
│  From Source                                                         │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────┐                                                    │
│  │  I/O Thread  │ ── Receives events from source                     │
│  └──────────────┘                                                    │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────┐                                                    │
│  │  Relay Log   │ ── Local copy of binary log events                 │
│  └──────────────┘                                                    │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────┐                                                    │
│  │  SQL Thread  │ ── Applies events to local database                │
│  └──────────────┘                                                    │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────┐                                                    │
│  │ Local Tables │                                                    │
│  └──────────────┘                                                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Replication Threads

### I/O Thread

```sql
-- Connects to source and reads binary log
-- Writes to local relay log

-- View I/O thread status
SHOW REPLICA STATUS\G
-- Look for:
-- Replica_IO_Running: Yes
-- Replica_IO_State: Waiting for source to send event

-- I/O thread creates:
-- relay-log.000001, relay-log.000002, ...
-- relay-log.index (list of relay logs)
```

### SQL Thread

```sql
-- Reads relay log and applies events

-- View SQL thread status
SHOW REPLICA STATUS\G
-- Look for:
-- Replica_SQL_Running: Yes
-- Replica_SQL_Running_State: Replica has read all relay log

-- Multi-threaded replication (MySQL 5.7+)
SHOW VARIABLES LIKE 'replica_parallel_workers';
SET GLOBAL replica_parallel_workers = 4;
```

### Binlog Dump Thread

```sql
-- Source-side thread that sends events to replicas

-- View connected replicas
SHOW PROCESSLIST;
-- Look for: Command = "Binlog Dump" or "Binlog Dump GTID"

-- Each replica connection has one binlog dump thread
```

---

## 7. Binary Log Maintenance

### Purging Old Logs

```sql
-- Automatic purge based on expiration
SET GLOBAL binlog_expire_logs_seconds = 604800;  -- 7 days

-- Manual purge
PURGE BINARY LOGS TO 'binlog.000010';
PURGE BINARY LOGS BEFORE '2024-01-15 00:00:00';

-- Reset (delete all binary logs)
RESET MASTER;  -- DANGEROUS! Breaks replication
```

### Binary Log Safety

```sql
-- Sync on every commit (safest)
sync_binlog = 1

-- Sync every N commits (faster, less safe)
sync_binlog = 100

-- Never sync (fastest, data loss risk)
sync_binlog = 0

-- Recommendation: Use sync_binlog = 1 for replication
```

---

## 8. Replication Filters

### Source-Side Filtering

```sql
-- my.cnf on source
[mysqld]
binlog_do_db = database1
binlog_do_db = database2
binlog_ignore_db = test

-- WARNING: Statement-based replication filter issues
-- Filter based on current database, not tables affected
```

### Replica-Side Filtering

```sql
-- my.cnf on replica
[mysqld]
replicate_do_db = database1
replicate_ignore_db = test
replicate_do_table = database1.users
replicate_ignore_table = database1.logs
replicate_wild_do_table = database1.user%

-- Runtime change (MySQL 8.0.22+)
CHANGE REPLICATION FILTER
    REPLICATE_DO_DB = (database1, database2),
    REPLICATE_IGNORE_DB = (test);
```

---

## Summary

| Concept | Description |
|---------|-------------|
| Binary Log | Records all database changes |
| SBR | Logs SQL statements |
| RBR | Logs row changes |
| GTID | Unique transaction identifier |
| I/O Thread | Receives events from source |
| SQL Thread | Applies events to replica |

---

## Further Reading

- MySQL Binary Log documentation
- GTID concepts documentation
- "High Performance MySQL" - Replication chapter
