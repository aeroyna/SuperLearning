# Monitoring and Diagnostics

## Learning Objectives
- Set up comprehensive MySQL monitoring
- Use built-in diagnostic tools
- Identify and resolve common issues
- Establish alerting for critical conditions

---

## 1. Essential Monitoring Metrics

### Key Performance Indicators

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MySQL Monitoring Dashboard                        │
│                                                                      │
│  QUERIES                          CONNECTIONS                        │
│  ┌─────────────────────────┐     ┌─────────────────────────┐        │
│  │ Queries/sec: 1,523      │     │ Current: 45/500         │        │
│  │ Slow queries: 12        │     │ Max used: 127           │        │
│  │ Questions: 2.3M         │     │ Aborted: 23             │        │
│  └─────────────────────────┘     └─────────────────────────┘        │
│                                                                      │
│  INNODB                           REPLICATION                        │
│  ┌─────────────────────────┐     ┌─────────────────────────┐        │
│  │ Buffer pool hit: 99.8%  │     │ Status: Running         │        │
│  │ Dirty pages: 234        │     │ Lag: 0 seconds          │        │
│  │ Pending I/O: 0          │     │ GTIDs behind: 0         │        │
│  └─────────────────────────┘     └─────────────────────────┘        │
│                                                                      │
│  STORAGE                          LOCKS                              │
│  ┌─────────────────────────┐     ┌─────────────────────────┐        │
│  │ Data size: 45.2 GB      │     │ Current locks: 12       │        │
│  │ Index size: 12.1 GB     │     │ Lock waits: 3           │        │
│  │ Free space: 234 GB      │     │ Deadlocks: 0            │        │
│  └─────────────────────────┘     └─────────────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘
```

### Critical Metrics

```sql
-- Query performance
SHOW GLOBAL STATUS LIKE 'Questions';
SHOW GLOBAL STATUS LIKE 'Slow_queries';
SHOW GLOBAL STATUS LIKE 'Com_select';
SHOW GLOBAL STATUS LIKE 'Com_insert';

-- Connections
SHOW GLOBAL STATUS LIKE 'Connections';
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Max_used_connections';
SHOW GLOBAL STATUS LIKE 'Aborted_connects';

-- InnoDB
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read_requests';
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_reads';
SHOW GLOBAL STATUS LIKE 'Innodb_row_lock_waits';
SHOW GLOBAL STATUS LIKE 'Innodb_deadlocks';

-- Temp tables
SHOW GLOBAL STATUS LIKE 'Created_tmp_tables';
SHOW GLOBAL STATUS LIKE 'Created_tmp_disk_tables';
```

---

## 2. Performance Schema

### Enable Performance Schema

```sql
-- Check if enabled
SHOW VARIABLES LIKE 'performance_schema';

-- Enable in my.cnf
[mysqld]
performance_schema = ON

-- Enable specific instruments
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES'
WHERE NAME LIKE 'statement/%';

-- Enable consumers
UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME LIKE 'events_statements%';
```

### Statement Analysis

```sql
-- Top queries by time
SELECT
    DIGEST_TEXT,
    COUNT_STAR AS exec_count,
    SUM_TIMER_WAIT/1000000000000 AS total_sec,
    AVG_TIMER_WAIT/1000000000 AS avg_ms,
    SUM_ROWS_EXAMINED,
    SUM_ROWS_SENT,
    SUM_NO_INDEX_USED
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Queries with full table scans
SELECT * FROM performance_schema.events_statements_summary_by_digest
WHERE SUM_NO_INDEX_USED > 0 OR SUM_NO_GOOD_INDEX_USED > 0
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

### Wait Analysis

```sql
-- Top wait events
SELECT
    EVENT_NAME,
    COUNT_STAR,
    SUM_TIMER_WAIT/1000000000000 AS total_sec
FROM performance_schema.events_waits_summary_global_by_event_name
WHERE COUNT_STAR > 0
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;

-- I/O waits by file
SELECT
    FILE_NAME,
    COUNT_READ,
    SUM_TIMER_READ/1000000000000 AS read_sec,
    COUNT_WRITE,
    SUM_TIMER_WRITE/1000000000000 AS write_sec
FROM performance_schema.file_summary_by_instance
ORDER BY SUM_TIMER_READ + SUM_TIMER_WRITE DESC
LIMIT 10;
```

---

## 3. sys Schema

### Quick Diagnostics

```sql
-- Statement analysis (user-friendly)
SELECT * FROM sys.statement_analysis LIMIT 10;

-- Statements with full table scans
SELECT * FROM sys.statements_with_full_table_scans LIMIT 10;

-- Statements with temp tables
SELECT * FROM sys.statements_with_temp_tables LIMIT 10;

-- Statements with sorting
SELECT * FROM sys.statements_with_sorting LIMIT 10;

-- Index usage
SELECT * FROM sys.schema_index_statistics LIMIT 10;

-- Unused indexes
SELECT * FROM sys.schema_unused_indexes;
```

### Host and User Analysis

```sql
-- Activity by host
SELECT * FROM sys.host_summary;

-- Activity by user
SELECT * FROM sys.user_summary;

-- Memory by user
SELECT * FROM sys.memory_by_user_by_current_bytes;

-- Memory by host
SELECT * FROM sys.memory_by_host_by_current_bytes;
```

### I/O Analysis

```sql
-- I/O by file
SELECT * FROM sys.io_global_by_file_by_latency LIMIT 10;

-- I/O by table
SELECT * FROM sys.schema_table_statistics_with_buffer
ORDER BY io_read + io_write DESC
LIMIT 10;

-- Wait analysis
SELECT * FROM sys.wait_classes_global_by_latency;
```

---

## 4. InnoDB Status

### SHOW ENGINE INNODB STATUS

```sql
SHOW ENGINE INNODB STATUS\G

-- Key sections to review:
-- SEMAPHORES: Lock/mutex contention
-- TRANSACTIONS: Active transactions, lock info
-- FILE I/O: I/O thread status
-- INSERT BUFFER AND ADAPTIVE HASH INDEX
-- LOG: Redo log activity
-- BUFFER POOL AND MEMORY: Buffer pool stats
-- ROW OPERATIONS: DML activity
```

### InnoDB Metrics

```sql
-- Enable InnoDB metrics
SET GLOBAL innodb_monitor_enable = 'all';

-- View metrics
SELECT NAME, COUNT, STATUS
FROM information_schema.INNODB_METRICS
WHERE STATUS = 'enabled'
ORDER BY NAME;

-- Key metrics
SELECT NAME, COUNT
FROM information_schema.INNODB_METRICS
WHERE NAME IN (
    'buffer_pool_reads',
    'buffer_pool_read_requests',
    'lock_deadlocks',
    'lock_timeouts',
    'log_writes',
    'trx_commits'
);
```

---

## 5. Monitoring Tools

### MySQL Enterprise Monitor

```sql
-- Agent-based monitoring
-- Includes Query Analyzer
-- Advisors for best practices
-- Alerting and notifications
```

### Percona Monitoring and Management (PMM)

```bash
# Install PMM Client
wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
dpkg -i percona-release_latest.generic_all.deb
apt-get update
apt-get install pmm2-client

# Register with PMM Server
pmm-admin config --server-url=https://admin:password@pmm.example.com

# Add MySQL monitoring
pmm-admin add mysql --username=pmm --password=xxx
```

### Prometheus and Grafana

```yaml
# mysqld_exporter for Prometheus
# prometheus.yml
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']

# Grafana dashboard IDs:
# 7362 - MySQL Overview
# 6239 - MySQL InnoDB Metrics
```

---

## 6. Alerting

### Critical Alerts

```sql
-- Connection exhaustion
SELECT
    @@max_connections AS max_conn,
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status
     WHERE VARIABLE_NAME = 'Threads_connected') AS current_conn,
    ((SELECT VARIABLE_VALUE FROM performance_schema.global_status
      WHERE VARIABLE_NAME = 'Threads_connected') / @@max_connections) * 100 AS pct_used;
-- Alert if > 80%

-- Replication lag
SHOW REPLICA STATUS\G
-- Alert if Seconds_Behind_Source > 30

-- Buffer pool hit ratio
-- Alert if < 99%

-- Slow queries
SHOW GLOBAL STATUS LIKE 'Slow_queries';
-- Alert if growing rapidly
```

### Monitoring Script

```bash
#!/bin/bash
# mysql_health_check.sh

MYSQL="mysql -u monitor -pxxx -N -B"

# Check connections
CONN_PCT=$($MYSQL -e "SELECT ROUND((@@max_connections -
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status
     WHERE VARIABLE_NAME='Threads_connected')) / @@max_connections * 100)")

if [ "$CONN_PCT" -lt 20 ]; then
    echo "CRITICAL: Only $CONN_PCT% connections available"
    exit 2
fi

# Check replication
REP_LAG=$($MYSQL -e "SHOW REPLICA STATUS\G" | grep "Seconds_Behind_Source" | awk '{print $2}')

if [ "$REP_LAG" == "NULL" ]; then
    echo "CRITICAL: Replication not running"
    exit 2
elif [ "$REP_LAG" -gt 30 ]; then
    echo "WARNING: Replication lag is $REP_LAG seconds"
    exit 1
fi

echo "OK: MySQL healthy"
exit 0
```

---

## 7. Troubleshooting

### High CPU Usage

```sql
-- Find long-running queries
SELECT
    ID,
    USER,
    HOST,
    DB,
    COMMAND,
    TIME,
    STATE,
    LEFT(INFO, 100) AS query
FROM information_schema.PROCESSLIST
WHERE COMMAND != 'Sleep'
ORDER BY TIME DESC;

-- Kill problematic query
KILL QUERY <id>;
KILL CONNECTION <id>;
```

### High Memory Usage

```sql
-- Check buffer pool
SELECT * FROM sys.memory_global_by_current_bytes
ORDER BY current_alloc DESC LIMIT 10;

-- Check per-connection memory
SELECT * FROM sys.memory_by_thread_by_current_bytes
ORDER BY current_alloc DESC LIMIT 10;

-- Reduce if needed
SET GLOBAL innodb_buffer_pool_size = 8G;
```

### Lock Contention

```sql
-- View current locks
SELECT * FROM performance_schema.data_locks;

-- View lock waits
SELECT * FROM performance_schema.data_lock_waits;

-- Find blocking queries
SELECT
    r.trx_id AS waiting_trx,
    r.trx_mysql_thread_id AS waiting_pid,
    r.trx_query AS waiting_query,
    b.trx_id AS blocking_trx,
    b.trx_mysql_thread_id AS blocking_pid,
    b.trx_query AS blocking_query
FROM information_schema.innodb_lock_waits w
JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id;
```

### Disk Space Issues

```sql
-- Table sizes
SELECT
    table_schema AS db,
    table_name,
    ROUND(data_length / 1024 / 1024, 2) AS data_mb,
    ROUND(index_length / 1024 / 1024, 2) AS index_mb,
    ROUND(data_free / 1024 / 1024, 2) AS free_mb
FROM information_schema.TABLES
ORDER BY data_length + index_length DESC
LIMIT 20;

-- Reclaim space
OPTIMIZE TABLE tablename;

-- Or rebuild
ALTER TABLE tablename ENGINE=InnoDB;
```

---

## 8. Log Analysis

### Error Log

```bash
# Location
SHOW VARIABLES LIKE 'log_error';

# View recent errors
tail -100 /var/log/mysql/error.log

# Search for specific issues
grep -i "error\|warning" /var/log/mysql/error.log | tail -50
```

### Slow Query Log

```bash
# Enable
SET GLOBAL slow_query_log = ON;
SET GLOBAL long_query_time = 2;

# Analyze
pt-query-digest /var/log/mysql/slow.log
```

### General Query Log (Debugging)

```sql
-- Enable temporarily (high overhead!)
SET GLOBAL general_log = ON;
SET GLOBAL general_log_file = '/tmp/queries.log';

-- Disable after debugging
SET GLOBAL general_log = OFF;
```

---

## 9. Health Check Queries

### Quick Health Check

```sql
-- Server status
SELECT
    @@hostname AS host,
    @@version AS version,
    @@uptime AS uptime_seconds;

-- Current activity
SELECT COUNT(*) AS active_queries
FROM information_schema.PROCESSLIST
WHERE COMMAND != 'Sleep';

-- Buffer pool
SELECT
    FORMAT((1 - (
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads') /
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests')
    )) * 100, 2) AS buffer_pool_hit_ratio;

-- Replication
SHOW REPLICA STATUS\G
```

### Comprehensive Health Report

```sql
-- Create health report procedure
DELIMITER //
CREATE PROCEDURE health_report()
BEGIN
    SELECT 'SERVER INFO' AS section;
    SELECT @@hostname, @@version, @@uptime;

    SELECT 'CONNECTIONS' AS section;
    SELECT
        @@max_connections AS max_conn,
        (SELECT VARIABLE_VALUE FROM performance_schema.global_status
         WHERE VARIABLE_NAME = 'Threads_connected') AS current;

    SELECT 'SLOW QUERIES' AS section;
    SELECT VARIABLE_VALUE AS slow_queries
    FROM performance_schema.global_status
    WHERE VARIABLE_NAME = 'Slow_queries';

    SELECT 'TOP TABLES BY SIZE' AS section;
    SELECT table_schema, table_name,
           ROUND((data_length + index_length) / 1024 / 1024, 2) AS size_mb
    FROM information_schema.TABLES
    ORDER BY data_length + index_length DESC
    LIMIT 5;
END //
DELIMITER ;

CALL health_report();
```

---

## Summary

| Tool | Use Case |
|------|----------|
| Performance Schema | Detailed performance data |
| sys Schema | User-friendly diagnostics |
| SHOW ENGINE INNODB STATUS | InnoDB internals |
| Error log | Server errors and warnings |
| Slow query log | Query performance issues |

---

## Further Reading

- MySQL Performance Schema documentation
- sys Schema documentation
- Percona PMM documentation
- "High Performance MySQL" - Monitoring chapter
