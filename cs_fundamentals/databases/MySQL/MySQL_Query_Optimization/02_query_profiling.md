# Query Profiling in MySQL

## Learning Objectives
- Master SHOW PROFILE and Performance Schema profiling
- Learn to identify query bottlenecks
- Understand query execution stages
- Use profiling to guide optimizations

---

## 1. SHOW PROFILE (Legacy)

### Enabling Profiling

```sql
-- Enable profiling for current session
SET profiling = 1;

-- Check status
SHOW VARIABLES LIKE 'profiling';

-- Set maximum profiles to keep
SET profiling_history_size = 100;
```

### Basic Profiling

```sql
-- Enable profiling
SET profiling = 1;

-- Run queries
SELECT * FROM orders WHERE status = 'pending';
SELECT COUNT(*) FROM orders GROUP BY user_id;
SELECT * FROM users WHERE email LIKE 'john%';

-- View query list
SHOW PROFILES;
+----------+------------+---------------------------------------------+
| Query_ID | Duration   | Query                                       |
+----------+------------+---------------------------------------------+
|        1 | 0.00215000 | SELECT * FROM orders WHERE status = 'pending' |
|        2 | 0.15234500 | SELECT COUNT(*) FROM orders GROUP BY user_id |
|        3 | 0.00089500 | SELECT * FROM users WHERE email LIKE 'john%' |
+----------+------------+---------------------------------------------+

-- Profile specific query
SHOW PROFILE FOR QUERY 2;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000075 |
| checking permissions | 0.000008 |
| Opening tables       | 0.000021 |
| init                 | 0.000018 |
| System lock          | 0.000011 |
| optimizing           | 0.000012 |
| statistics           | 0.000089 |
| preparing            | 0.000015 |
| Creating tmp table   | 0.002345 |
| executing            | 0.145623 |
| end                  | 0.000012 |
| removing tmp tables  | 0.000015 |
| query end            | 0.000008 |
| closing tables       | 0.000012 |
| freeing items        | 0.000045 |
| cleaning up          | 0.000015 |
+----------------------+----------+
```

### Detailed Profile Options

```sql
-- Profile with CPU info
SHOW PROFILE CPU FOR QUERY 2;
+----------------------+----------+----------+------------+
| Status               | Duration | CPU_user | CPU_system |
+----------------------+----------+----------+------------+
| executing            | 0.145623 | 0.140000 | 0.004000   |
+----------------------+----------+----------+------------+

-- Profile with all information
SHOW PROFILE ALL FOR QUERY 2;

-- Available profile types:
-- ALL: All information
-- BLOCK IO: Block input/output operations
-- CONTEXT SWITCHES: Context switches
-- CPU: CPU usage
-- IPC: Interprocess communication
-- MEMORY: Memory usage (not implemented)
-- PAGE FAULTS: Page faults
-- SOURCE: Source code location
-- SWAPS: Swap operations
```

---

## 2. Performance Schema Profiling

### Statement Analysis

```sql
-- Enable statement instrumentation (usually on by default)
UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME LIKE 'events_statements%';

-- Get top queries by total time
SELECT
    DIGEST_TEXT,
    COUNT_STAR AS exec_count,
    SUM_TIMER_WAIT/1000000000000 AS total_sec,
    AVG_TIMER_WAIT/1000000000 AS avg_ms,
    MAX_TIMER_WAIT/1000000000 AS max_ms,
    SUM_ROWS_EXAMINED AS rows_examined,
    SUM_ROWS_SENT AS rows_sent
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

### Stage Profiling

```sql
-- Enable stage monitoring
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES'
WHERE NAME LIKE 'stage/%';

UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME LIKE 'events_stages%';

-- Run a query
SELECT * FROM orders WHERE status = 'pending';

-- View stages for recent statement
SELECT
    EVENT_NAME,
    TIMER_WAIT/1000000000 AS duration_ms,
    SOURCE
FROM performance_schema.events_stages_history_long
WHERE NESTING_EVENT_ID = (
    SELECT EVENT_ID
    FROM performance_schema.events_statements_history_long
    ORDER BY TIMER_START DESC LIMIT 1
)
ORDER BY TIMER_START;
```

### Wait Event Analysis

```sql
-- Enable wait event monitoring
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES'
WHERE NAME LIKE 'wait/%';

UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME LIKE 'events_waits%';

-- Top wait events globally
SELECT
    EVENT_NAME,
    COUNT_STAR,
    SUM_TIMER_WAIT/1000000000000 AS total_seconds
FROM performance_schema.events_waits_summary_global_by_event_name
WHERE COUNT_STAR > 0
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;

-- Common wait types:
-- wait/io/file/* - Disk I/O
-- wait/synch/mutex/* - Lock contention
-- wait/lock/table/* - Table locks
```

---

## 3. sys Schema Profiling

### Statement Analysis Views

```sql
-- Top statements by latency
SELECT * FROM sys.statement_analysis LIMIT 10;

-- Statements with full table scans
SELECT * FROM sys.statements_with_full_table_scans LIMIT 10;

-- Statements with temporary tables
SELECT * FROM sys.statements_with_temp_tables LIMIT 10;

-- Statements with sorting
SELECT * FROM sys.statements_with_sorting LIMIT 10;

-- Statements with errors/warnings
SELECT * FROM sys.statements_with_errors_or_warnings LIMIT 10;
```

### Host and User Analysis

```sql
-- Activity by host
SELECT * FROM sys.host_summary;

-- Activity by user
SELECT * FROM sys.user_summary;

-- Statement latency by user
SELECT * FROM sys.user_summary_by_statement_latency;
```

### I/O Analysis

```sql
-- I/O by file
SELECT * FROM sys.io_global_by_file_by_latency LIMIT 10;

-- I/O by table
SELECT * FROM sys.schema_table_statistics_with_buffer LIMIT 10;

-- Index usage
SELECT * FROM sys.schema_index_statistics LIMIT 10;
```

---

## 4. Query Execution Stages

### Understanding Stages

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Query Execution Stages                            │
│                                                                      │
│  Stage                      | What Happens                          │
│  ─────────────────────────────────────────────────────────────────  │
│  starting                   | Initialize query execution            │
│  checking permissions       | Verify user privileges                │
│  Opening tables             | Open and lock tables                  │
│  init                       | Initialize query structures           │
│  System lock                | Acquire system locks                  │
│  optimizing                 | Query optimizer runs                  │
│  statistics                 | Gather index statistics               │
│  preparing                  | Prepare execution plan                │
│  Creating tmp table         | Create temporary table (if needed)   │
│  executing                  | Execute the query                     │
│  Sending data               | Send results to client               │
│  Sorting result             | Sort results (filesort)               │
│  end                        | Query execution complete              │
│  removing tmp tables        | Clean up temporary tables             │
│  closing tables             | Close and unlock tables               │
│  freeing items              | Free memory                           │
│  cleaning up                | Final cleanup                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Bottleneck Identification

```sql
-- If most time in "executing":
-- - Check for full table scans
-- - Add or optimize indexes
-- - Rewrite query

-- If most time in "Sending data":
-- - Large result set
-- - Network issues
-- - Client processing slow

-- If "Creating tmp table" is slow:
-- - Query needs temp table
-- - Consider increasing tmp_table_size
-- - Rewrite to avoid temp tables

-- If "Sorting result" is slow:
-- - No suitable index for ORDER BY
-- - Add index to support sort
-- - Increase sort_buffer_size
```

---

## 5. Real-Time Monitoring

### Current Queries

```sql
-- Show running queries
SHOW PROCESSLIST;

-- Full query text
SHOW FULL PROCESSLIST;

-- Using Performance Schema
SELECT
    THREAD_ID,
    USER,
    DB,
    COMMAND,
    TIME,
    STATE,
    LEFT(INFO, 100) AS query_prefix
FROM performance_schema.threads
JOIN information_schema.PROCESSLIST USING (PROCESSLIST_ID)
WHERE COMMAND != 'Sleep'
ORDER BY TIME DESC;
```

### Query Progress (MySQL 8.0)

```sql
-- Monitor query progress
SELECT
    PS.THREAD_ID,
    PS.SQL_TEXT,
    STAGE.EVENT_NAME AS current_stage,
    STAGE.WORK_COMPLETED,
    STAGE.WORK_ESTIMATED,
    ROUND((STAGE.WORK_COMPLETED / STAGE.WORK_ESTIMATED) * 100, 2) AS pct_complete
FROM performance_schema.events_statements_current PS
JOIN performance_schema.events_stages_current STAGE
    ON STAGE.THREAD_ID = PS.THREAD_ID
WHERE PS.SQL_TEXT IS NOT NULL;
```

---

## 6. Optimizer Trace

### Detailed Optimization Analysis

```sql
-- Enable optimizer trace
SET optimizer_trace = 'enabled=on';

-- Run query
SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at;

-- View trace
SELECT * FROM information_schema.OPTIMIZER_TRACE\G

-- Disable
SET optimizer_trace = 'enabled=off';
```

### Reading Optimizer Trace

```sql
-- Trace shows:
-- 1. join_preparation: Query parsing and normalization
-- 2. join_optimization: Cost-based optimization decisions
-- 3. join_execution: Execution plan details

-- Look for:
-- - "considered_execution_plans": Alternative plans evaluated
-- - "best_access_path": Why specific access method chosen
-- - "attached_conditions_computation": Filter pushdown decisions
```

---

## 7. Benchmark Queries

### Simple Timing

```sql
-- Basic timing
SET @start = NOW(6);
SELECT * FROM orders WHERE status = 'pending';
SELECT TIMEDIFF(NOW(6), @start) AS execution_time;

-- Using BENCHMARK for repeated execution
SELECT BENCHMARK(1000, (SELECT COUNT(*) FROM users));
-- Returns total time for 1000 iterations
```

### Statistical Profiling

```sql
-- Run query multiple times and gather statistics
DELIMITER //
CREATE PROCEDURE profile_query(IN p_iterations INT)
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE total_time DECIMAL(20,6) DEFAULT 0;
    DECLARE start_time DATETIME(6);
    DECLARE end_time DATETIME(6);
    DECLARE min_time DECIMAL(20,6) DEFAULT 999999;
    DECLARE max_time DECIMAL(20,6) DEFAULT 0;
    DECLARE current_time DECIMAL(20,6);

    WHILE i < p_iterations DO
        SET start_time = NOW(6);

        -- Your query here
        SELECT COUNT(*) INTO @dummy FROM orders WHERE status = 'pending';

        SET end_time = NOW(6);
        SET current_time = TIMESTAMPDIFF(MICROSECOND, start_time, end_time) / 1000000;
        SET total_time = total_time + current_time;
        IF current_time < min_time THEN SET min_time = current_time; END IF;
        IF current_time > max_time THEN SET max_time = current_time; END IF;
        SET i = i + 1;
    END WHILE;

    SELECT
        p_iterations AS iterations,
        total_time AS total_seconds,
        total_time / p_iterations AS avg_seconds,
        min_time AS min_seconds,
        max_time AS max_seconds;
END //
DELIMITER ;

CALL profile_query(100);
```

---

## 8. Profiling Best Practices

### Before Profiling

```sql
-- Clear query cache (if using MySQL < 8.0)
RESET QUERY CACHE;

-- Clear buffer pool state
-- (Only in development/testing!)
SET GLOBAL innodb_buffer_pool_dump_at_shutdown = 0;
-- Restart MySQL

-- Ensure consistent state
FLUSH TABLES;
```

### During Profiling

```sql
-- Run query multiple times
-- First run often slower (cold cache)
SELECT * FROM orders WHERE status = 'pending';  -- Cold
SELECT * FROM orders WHERE status = 'pending';  -- Warm
SELECT * FROM orders WHERE status = 'pending';  -- Warm

-- Compare warm-cache performance
```

### Interpreting Results

```sql
-- Compare before/after optimization
-- Document:
-- 1. Original query time
-- 2. Optimization applied
-- 3. New query time
-- 4. Improvement percentage

-- Example:
-- Before: 1.5 seconds
-- After (added index): 0.02 seconds
-- Improvement: 98.7%
```

---

## Summary

| Tool | Best For | Availability |
|------|----------|--------------|
| SHOW PROFILE | Quick single query analysis | Deprecated, still works |
| Performance Schema | Detailed system-wide analysis | MySQL 5.5+ |
| sys Schema | User-friendly views | MySQL 5.7+ |
| Optimizer Trace | Understanding optimizer decisions | MySQL 5.6+ |
| EXPLAIN ANALYZE | Actual vs estimated metrics | MySQL 8.0.18+ |

### Key Metrics to Watch

1. **Execution time** - Total query duration
2. **Rows examined** - How much data scanned
3. **Temporary tables** - Memory/disk temp usage
4. **Filesort** - Additional sorting operations
5. **Wait events** - Lock contention, I/O waits

---

## Further Reading

- MySQL Performance Schema documentation
- sys Schema views reference
- "High Performance MySQL" - Profiling chapter
