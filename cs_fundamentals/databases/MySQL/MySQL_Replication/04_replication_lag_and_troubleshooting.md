# Replication Lag and Troubleshooting

## Learning Objectives
- Understand causes of replication lag
- Monitor and measure replication health
- Diagnose common replication problems
- Apply solutions and preventive measures

---

## 1. Understanding Replication Lag

### What is Replication Lag?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Replication Lag Illustrated                       │
│                                                                      │
│  Time ─────────────────────────────────────────────────────────────► │
│                                                                      │
│  Source:  [Tx1][Tx2][Tx3][Tx4][Tx5][Tx6][Tx7][Tx8][Tx9][Tx10]       │
│                                                                      │
│  Replica: [Tx1][Tx2][Tx3][Tx4][Tx5]                                 │
│                                   ↑                                  │
│                                   │                                  │
│                              Lag = 5 transactions                    │
│                              (or measured in seconds)                │
│                                                                      │
│  Causes:                                                             │
│  • Source too fast                                                   │
│  • Replica too slow                                                  │
│  • Network delays                                                    │
│  • Long-running transactions                                         │
│  • Single-threaded SQL thread                                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Measuring Lag

```sql
-- Basic lag measurement
SHOW REPLICA STATUS\G

-- Key field:
-- Seconds_Behind_Source: Lag in seconds

-- Note: This can be misleading!
-- It measures time difference between event timestamp and current time
-- Not actual replication delay in all cases
```

### More Accurate Lag Measurement

```sql
-- Using heartbeat tables (Percona Toolkit approach)
CREATE TABLE heartbeat (
    ts TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    server_id INT UNSIGNED NOT NULL PRIMARY KEY
);

-- On source: Update every second
UPDATE heartbeat SET ts = NOW() WHERE server_id = 1;

-- On replica: Check actual lag
SELECT TIMESTAMPDIFF(SECOND, ts, NOW()) AS lag_seconds
FROM heartbeat WHERE server_id = 1;

-- Or use pt-heartbeat
pt-heartbeat --update --database=test --create-table
pt-heartbeat --monitor --database=test
```

---

## 2. Common Causes of Lag

### 1. Single-Threaded Applier

```sql
-- Problem: SQL thread applies sequentially
-- Source has parallel writes, replica applies one-by-one

-- Solution: Multi-threaded replication
SET GLOBAL replica_parallel_workers = 4;
SET GLOBAL replica_parallel_type = 'LOGICAL_CLOCK';
-- or 'DATABASE' for per-database parallelism

-- Configuration
[mysqld]
replica_parallel_workers = 4
replica_parallel_type = LOGICAL_CLOCK
replica_preserve_commit_order = ON
```

### 2. Large Transactions

```sql
-- Problem: Single large transaction blocks applier
DELETE FROM logs WHERE created_at < '2023-01-01';  -- Millions of rows

-- Solution: Batch operations
WHILE (remaining > 0) DO
    DELETE FROM logs WHERE created_at < '2023-01-01' LIMIT 10000;
    SELECT SLEEP(0.1);  -- Brief pause to let replica catch up
END WHILE;
```

### 3. Schema Changes (DDL)

```sql
-- Problem: ALTER TABLE blocks replication
ALTER TABLE large_table ADD COLUMN new_col VARCHAR(100);

-- Solution: Use online DDL
ALTER TABLE large_table ADD COLUMN new_col VARCHAR(100),
    ALGORITHM = INPLACE,
    LOCK = NONE;

-- Or use pt-online-schema-change
pt-online-schema-change --alter "ADD COLUMN new_col VARCHAR(100)" \
    D=mydb,t=large_table --execute
```

### 4. Network Issues

```sql
-- Check network performance
SHOW REPLICA STATUS\G
-- High Seconds_Behind_Source with Replica_IO_Running: Yes
-- May indicate network delays

-- Monitor I/O thread lag
SELECT
    RECEIVED_TRANSACTION_SET,
    LAST_QUEUED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP
FROM performance_schema.replication_connection_status;
```

### 5. Slow Disk I/O

```sql
-- Replica disk slower than source

-- Solutions:
-- 1. Better storage (SSD)
-- 2. Increase InnoDB buffer pool
SET GLOBAL innodb_buffer_pool_size = 8G;

-- 3. Tune I/O settings
innodb_flush_method = O_DIRECT
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000
```

---

## 3. Monitoring Replication

### Performance Schema

```sql
-- Applier status
SELECT * FROM performance_schema.replication_applier_status\G

-- Applier worker status
SELECT
    WORKER_ID,
    LAST_SEEN_TRANSACTION,
    LAST_ERROR_NUMBER,
    LAST_ERROR_MESSAGE,
    LAST_ERROR_TIMESTAMP
FROM performance_schema.replication_applier_status_by_worker;

-- Connection status
SELECT * FROM performance_schema.replication_connection_status\G

-- Detailed lag by GTID
SELECT
    SOURCE_UUID,
    TRANSACTION_NAME,
    APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP,
    TIMESTAMPDIFF(SECOND, APPLYING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP, NOW()) AS lag_seconds
FROM performance_schema.replication_applier_status_by_coordinator;
```

### Alerting Thresholds

```sql
-- Create monitoring view
CREATE VIEW replication_health AS
SELECT
    CASE
        WHEN SBS.SERVICE_STATE != 'ON' THEN 'CRITICAL'
        WHEN SBS.SERVICE_STATE = 'ON' AND
             (TIMESTAMPDIFF(SECOND, NOW(),
              (SELECT MAX(ts) FROM heartbeat)) > 60) THEN 'WARNING'
        ELSE 'OK'
    END AS health_status,
    SBS.SERVICE_STATE AS sql_thread_state,
    SBC.SERVICE_STATE AS io_thread_state
FROM performance_schema.replication_applier_status SBS
JOIN performance_schema.replication_connection_status SBC
ON SBS.CHANNEL_NAME = SBC.CHANNEL_NAME;
```

### Monitoring Script

```bash
#!/bin/bash
# check_replication.sh

LAG_THRESHOLD=30
RESULT=$(mysql -N -e "SHOW REPLICA STATUS\G" | grep "Seconds_Behind_Source" | awk '{print $2}')

if [ "$RESULT" == "NULL" ]; then
    echo "CRITICAL: Replication not running"
    exit 2
elif [ "$RESULT" -gt "$LAG_THRESHOLD" ]; then
    echo "WARNING: Replication lag is ${RESULT} seconds"
    exit 1
else
    echo "OK: Replication lag is ${RESULT} seconds"
    exit 0
fi
```

---

## 4. Troubleshooting Common Issues

### Replication Stopped - Duplicate Entry

```sql
-- Error: Duplicate entry '123' for key 'PRIMARY'

-- Option 1: Skip the transaction (use cautiously!)
-- With GTID:
SET GTID_NEXT = 'source_uuid:transaction_id';
BEGIN; COMMIT;
SET GTID_NEXT = 'AUTOMATIC';
START REPLICA;

-- Without GTID:
SET GLOBAL sql_replica_skip_counter = 1;
START REPLICA;

-- Option 2: Find and fix the inconsistency
pt-table-checksum --replicate=test.checksums h=source
pt-table-sync --sync-to-master h=replica
```

### Replication Stopped - Missing Table

```sql
-- Error: Table 'mydb.users' doesn't exist

-- Check if table exists on source
-- If table was dropped on replica accidentally:

-- 1. Stop replication
STOP REPLICA;

-- 2. Get table structure from source
mysqldump -h source --no-data mydb users | mysql

-- 3. Resume replication
START REPLICA;
```

### I/O Thread Not Running

```sql
-- Error: Replica_IO_Running: No

-- Check error message
SHOW REPLICA STATUS\G
-- Look at: Last_IO_Error

-- Common causes:
-- 1. Wrong credentials
-- 2. Network connectivity
-- 3. Source binary logs purged

-- Fix: Reset and reconfigure
STOP REPLICA;
CHANGE REPLICATION SOURCE TO
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'new_password';
START REPLICA;
```

### SQL Thread Not Running

```sql
-- Error: Replica_SQL_Running: No

-- Check error message
SHOW REPLICA STATUS\G
-- Look at: Last_SQL_Error

-- Common causes:
-- 1. Data inconsistency
-- 2. Missing table/database
-- 3. Constraint violation

-- Debug: Try applying manually
-- Find the failed statement in relay log
mysqlbinlog relay-bin.000001 | less
```

### Binary Log Corrupted/Purged

```sql
-- Error: Could not find first log file name in binary log index

-- Solution 1: Fresh start with GTID
STOP REPLICA;
RESET REPLICA ALL;

-- Clone data from source
CLONE INSTANCE FROM 'repl'@'source':3306 IDENTIFIED BY 'password';

-- After restart, configure and start
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'source',
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'password',
    SOURCE_AUTO_POSITION = 1;
START REPLICA;

-- Solution 2: Rebuild from backup
-- 1. Stop replica MySQL
-- 2. Restore backup
-- 3. Configure replication from backup position
```

---

## 5. Fixing Data Inconsistencies

### Using pt-table-checksum

```bash
# On source: Check for differences
pt-table-checksum \
    --replicate=percona.checksums \
    --databases=mydb \
    h=source,u=root,p=xxx

# View differences
pt-table-checksum \
    --replicate=percona.checksums \
    --replicate-check-only \
    h=source,u=root,p=xxx
```

### Using pt-table-sync

```bash
# Preview changes
pt-table-sync \
    --sync-to-master \
    --print \
    h=replica,u=root,p=xxx

# Apply fixes
pt-table-sync \
    --sync-to-master \
    --execute \
    h=replica,u=root,p=xxx
```

### Manual Fix

```sql
-- Find inconsistent rows
-- On source:
SELECT * FROM users WHERE id = 123;

-- On replica:
SELECT * FROM users WHERE id = 123;

-- Fix on replica (if allowed):
SET sql_log_bin = 0;  -- Don't replicate this fix
INSERT/UPDATE/DELETE as needed;
SET sql_log_bin = 1;
```

---

## 6. Preventive Measures

### Configuration Best Practices

```ini
[mysqld]
# Multi-threaded replication
replica_parallel_workers = 4
replica_parallel_type = LOGICAL_CLOCK
replica_preserve_commit_order = ON

# Crash-safe replication
relay_log_recovery = ON
relay_log_info_repository = TABLE
master_info_repository = TABLE

# Sync settings
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

# Retention
binlog_expire_logs_seconds = 604800
```

### Application Best Practices

```sql
-- 1. Use transactions
BEGIN;
INSERT INTO orders VALUES (...);
INSERT INTO order_items VALUES (...);
COMMIT;

-- 2. Batch large operations
-- Instead of:
DELETE FROM logs WHERE date < '2023-01-01';

-- Use:
DELETE FROM logs WHERE date < '2023-01-01' LIMIT 10000;
-- Repeat until done

-- 3. Monitor before reads after writes
-- If reading from replica after write:
-- - Accept stale reads, or
-- - Read from source for critical reads
```

### Monitoring Checklist

```markdown
1. [ ] Seconds_Behind_Source (alert > 30s)
2. [ ] Replica_IO_Running = Yes
3. [ ] Replica_SQL_Running = Yes
4. [ ] Last_Error fields empty
5. [ ] pt-heartbeat lag
6. [ ] pt-table-checksum weekly
```

---

## Summary

| Issue | Quick Fix | Proper Fix |
|-------|-----------|------------|
| Lag | Increase workers | Optimize source, network |
| Duplicate entry | Skip transaction | Fix source of duplicates |
| Missing table | Create from source | pt-table-sync |
| I/O stopped | Check credentials | Fix connectivity |
| SQL stopped | Skip or fix | Investigate root cause |

---

## Further Reading

- MySQL Replication troubleshooting guide
- Percona Toolkit documentation
- "High Performance MySQL" - Replication chapter
