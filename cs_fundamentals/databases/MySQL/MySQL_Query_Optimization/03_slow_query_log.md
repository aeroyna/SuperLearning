# Slow Query Log

## Learning Objectives
- Configure and manage the slow query log
- Analyze slow queries effectively
- Use tools like pt-query-digest
- Establish continuous monitoring practices

---

## 1. Slow Query Log Configuration

### Basic Configuration

```sql
-- View current settings
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
SHOW VARIABLES LIKE 'log_queries_not_using_indexes';

-- Enable slow query log
SET GLOBAL slow_query_log = 1;
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
SET GLOBAL long_query_time = 1;  -- Queries taking > 1 second

-- Log queries not using indexes
SET GLOBAL log_queries_not_using_indexes = 1;

-- Limit logging rate for queries not using indexes
SET GLOBAL log_throttle_queries_not_using_indexes = 60;  -- Max 60/minute
```

### Configuration File (my.cnf)

```ini
[mysqld]
# Enable slow query log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1

# Log queries not using indexes
log_queries_not_using_indexes = 1
log_throttle_queries_not_using_indexes = 60

# Additional options
log_slow_admin_statements = 1    # Log slow admin commands
log_slow_slave_statements = 1    # Log slow replication statements
min_examined_row_limit = 100     # Only log if examining > 100 rows
```

### Logging to Table

```sql
-- Log to table instead of file
SET GLOBAL log_output = 'TABLE';
-- Or both
SET GLOBAL log_output = 'TABLE,FILE';

-- Query the slow log table
SELECT * FROM mysql.slow_log
ORDER BY start_time DESC
LIMIT 10;

-- Truncate old entries
TRUNCATE mysql.slow_log;
```

---

## 2. Understanding Slow Log Entries

### Log Entry Format

```
# Time: 2024-01-15T10:30:45.123456Z
# User@Host: app_user[app_user] @ web_server [192.168.1.100]  Id: 12345
# Query_time: 5.234567  Lock_time: 0.000123  Rows_sent: 1000  Rows_examined: 500000
SET timestamp=1705315845;
SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at DESC;
```

### Entry Fields Explained

| Field | Description |
|-------|-------------|
| Time | When query finished |
| User@Host | Who executed, from where |
| Query_time | Total execution time (seconds) |
| Lock_time | Time waiting for locks |
| Rows_sent | Rows returned to client |
| Rows_examined | Rows scanned (high = inefficient) |
| timestamp | Unix timestamp |

### Key Ratios to Monitor

```sql
-- Efficiency ratio: Rows_examined / Rows_sent
-- Good: < 10
-- Bad: > 100
-- Critical: > 1000

-- Example analysis:
-- Rows_sent: 10, Rows_examined: 100000
-- Ratio: 10000:1 - Very inefficient!
-- This query scans 10,000 rows for every 1 returned
```

---

## 3. Analyzing Slow Logs

### Manual Analysis

```bash
# View recent slow queries
tail -100 /var/log/mysql/slow.log

# Count slow queries per hour
grep "^# Time:" /var/log/mysql/slow.log | \
  cut -d'T' -f1,2 | cut -d':' -f1-2 | \
  sort | uniq -c

# Find queries with high rows_examined
grep -A 2 "Rows_examined: [0-9]\{6,\}" /var/log/mysql/slow.log
```

### mysqldumpslow

```bash
# Built-in tool for slow log analysis

# Sort by count
mysqldumpslow -s c /var/log/mysql/slow.log

# Sort by total time
mysqldumpslow -s t /var/log/mysql/slow.log

# Sort by average time
mysqldumpslow -s at /var/log/mysql/slow.log

# Sort by rows examined
mysqldumpslow -s ar /var/log/mysql/slow.log

# Top 10 queries
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log

# Show actual values instead of abstracting
mysqldumpslow -a /var/log/mysql/slow.log
```

### Output Example

```
Count: 1523  Time=5.32s (8102s)  Lock=0.00s (0s)  Rows=100.0 (152300), app_user
  SELECT * FROM orders WHERE status = 'S' ORDER BY created_at DESC LIMIT N

Count: 892  Time=2.15s (1917s)  Lock=0.00s (0s)  Rows=1.0 (892), app_user
  SELECT COUNT(*) FROM products WHERE category_id = N AND in_stock = N
```

---

## 4. pt-query-digest (Percona Toolkit)

### Installation

```bash
# Install Percona Toolkit
# Ubuntu/Debian
apt-get install percona-toolkit

# CentOS/RHEL
yum install percona-toolkit

# macOS
brew install percona-toolkit
```

### Basic Usage

```bash
# Analyze slow log
pt-query-digest /var/log/mysql/slow.log

# Limit to top 10 queries
pt-query-digest --limit=10 /var/log/mysql/slow.log

# Filter by user
pt-query-digest --filter '$event->{user} eq "app_user"' /var/log/mysql/slow.log

# Filter by database
pt-query-digest --filter '$event->{db} eq "production"' /var/log/mysql/slow.log

# Time range
pt-query-digest --since '2024-01-15 00:00:00' --until '2024-01-15 12:00:00' /var/log/mysql/slow.log
```

### Report Output

```
# Profile
# Rank Query ID                        Response time  Calls R/Call V/M
# ==== =============================== ============== ===== ====== ===
#    1 0x1B2B3C4D5E6F7A8B9C0D1E2F3A4B5 5000.0000 45.0%  1523 3.2840  0.12
#    2 0x2C3D4E5F6A7B8C9D0E1F2A3B4C5D6 2500.0000 22.5%   892 2.8027  0.08
#    3 0x3D4E5F6A7B8C9D0E1F2A3B4C5D6E7 1200.0000 10.8%   432 2.7778  0.15

# Query 1: 0x1B2B3C4D5E6F7A8B9C0D1E2F3A4B5
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         45    1523
# Exec time     45   5000s   500ms      8s      3s      6s   750ms      3s
# Lock time      1    10ms    10us   100us    20us    50us     5us    15us
# Rows sent     15 152.30k       1     200     100     150      30     100
# Rows examine  85  75.00M  20000  100000   50000   80000   15000   50000
# Query size    12   2.00M   1000    2000    1300    1800     200    1200

SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at DESC LIMIT N
```

### Advanced Options

```bash
# Save to file
pt-query-digest /var/log/mysql/slow.log > report.txt

# Review history (track trends over time)
pt-query-digest --history h=localhost,D=percona,t=query_history /var/log/mysql/slow.log

# Compare two time periods
pt-query-digest /var/log/mysql/slow.log.1 > before.txt
pt-query-digest /var/log/mysql/slow.log.2 > after.txt
diff before.txt after.txt

# Output as JSON
pt-query-digest --output json /var/log/mysql/slow.log

# Explain queries
pt-query-digest --explain h=localhost /var/log/mysql/slow.log
```

---

## 5. Performance Schema Alternative

### Statement History

```sql
-- Enable statement history
UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES'
WHERE NAME IN (
    'events_statements_history',
    'events_statements_history_long'
);

-- View recent slow statements
SELECT
    THREAD_ID,
    SQL_TEXT,
    TIMER_WAIT/1000000000000 AS seconds,
    ROWS_EXAMINED,
    ROWS_SENT,
    CREATED_TMP_DISK_TABLES,
    NO_INDEX_USED
FROM performance_schema.events_statements_history_long
WHERE TIMER_WAIT > 1000000000000  -- > 1 second
ORDER BY TIMER_WAIT DESC
LIMIT 20;
```

### Aggregated Statistics

```sql
-- Top queries by total time
SELECT
    DIGEST_TEXT,
    COUNT_STAR AS executions,
    SUM_TIMER_WAIT/1000000000000 AS total_seconds,
    AVG_TIMER_WAIT/1000000000 AS avg_ms,
    SUM_ROWS_EXAMINED,
    SUM_ROWS_SENT,
    SUM_ROWS_EXAMINED / NULLIF(SUM_ROWS_SENT, 0) AS exam_per_sent,
    SUM_NO_INDEX_USED AS no_index_count
FROM performance_schema.events_statements_summary_by_digest
WHERE SCHEMA_NAME = 'production'
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

---

## 6. Automated Monitoring

### Continuous Analysis Script

```bash
#!/bin/bash
# analyze_slow_queries.sh

LOG_FILE="/var/log/mysql/slow.log"
REPORT_DIR="/var/reports/mysql"
DATE=$(date +%Y%m%d)

# Rotate log
mysqladmin flush-logs

# Analyze previous log
pt-query-digest ${LOG_FILE}.1 > ${REPORT_DIR}/slow_query_${DATE}.txt

# Extract summary for alerting
SLOW_COUNT=$(grep -c "^# Time:" ${LOG_FILE}.1)
CRITICAL_COUNT=$(grep -c "Query_time: [0-9]\{2,\}" ${LOG_FILE}.1)

if [ $CRITICAL_COUNT -gt 10 ]; then
    mail -s "Critical: ${CRITICAL_COUNT} very slow queries" dba@company.com < ${REPORT_DIR}/slow_query_${DATE}.txt
fi

# Cleanup old reports
find ${REPORT_DIR} -name "slow_query_*.txt" -mtime +30 -delete
```

### Monitoring Thresholds

```sql
-- Create monitoring view
CREATE VIEW slow_query_stats AS
SELECT
    DATE(start_time) AS query_date,
    COUNT(*) AS slow_count,
    AVG(query_time) AS avg_time,
    MAX(query_time) AS max_time,
    SUM(rows_examined) AS total_examined
FROM mysql.slow_log
GROUP BY DATE(start_time);

-- Alert query
SELECT * FROM slow_query_stats
WHERE query_date = CURDATE()
  AND (slow_count > 100 OR max_time > 30);
```

---

## 7. Best Practices

### Log Rotation

```bash
# logrotate configuration
# /etc/logrotate.d/mysql-slow

/var/log/mysql/slow.log {
    daily
    rotate 7
    missingok
    notifempty
    compress
    delaycompress
    postrotate
        mysqladmin flush-logs
    endscript
}
```

### Threshold Strategy

```sql
-- Development: Aggressive logging
SET GLOBAL long_query_time = 0.1;  -- 100ms
SET GLOBAL log_queries_not_using_indexes = 1;

-- Staging: Moderate
SET GLOBAL long_query_time = 0.5;  -- 500ms
SET GLOBAL log_queries_not_using_indexes = 1;

-- Production: Conservative
SET GLOBAL long_query_time = 2;    -- 2 seconds
SET GLOBAL log_queries_not_using_indexes = 0;  -- Too noisy
SET GLOBAL log_throttle_queries_not_using_indexes = 10;
```

### Analysis Workflow

```markdown
1. Weekly Review
   - Run pt-query-digest on weekly logs
   - Identify top 10 slow queries
   - Create tickets for optimization

2. Immediate Response
   - Monitor for queries > 30 seconds
   - Kill if blocking other queries
   - Root cause analysis

3. Trend Tracking
   - Compare weekly reports
   - Track query count trends
   - Monitor for regressions after deployments
```

---

## 8. Troubleshooting Common Issues

### Log Not Writing

```sql
-- Check permissions
ls -la /var/log/mysql/

-- Check configuration
SHOW VARIABLES LIKE 'slow_query_log%';

-- Check if logging is actually enabled
SHOW GLOBAL STATUS LIKE 'Slow_queries';

-- Test with a deliberately slow query
SELECT SLEEP(5);
```

### Log Growing Too Fast

```sql
-- Increase threshold
SET GLOBAL long_query_time = 5;

-- Reduce index-missing logging
SET GLOBAL log_queries_not_using_indexes = 0;

-- Set minimum rows examined
SET GLOBAL min_examined_row_limit = 1000;
```

### Identifying Query Source

```sql
-- Enable query comments
-- Application should add comments like:
-- /* app=web, endpoint=/api/orders */ SELECT * FROM orders

-- Then in slow log analysis:
grep "app=web" /var/log/mysql/slow.log | head
```

---

## Summary

| Tool | Best For | Features |
|------|----------|----------|
| Slow Query Log | Capturing slow queries | Native, always available |
| mysqldumpslow | Quick analysis | Built-in, simple |
| pt-query-digest | Comprehensive analysis | Advanced reporting, trending |
| Performance Schema | Real-time monitoring | No file I/O, aggregated stats |

### Key Takeaways

1. **Enable slow query logging** in all environments
2. **Set appropriate thresholds** for each environment
3. **Use pt-query-digest** for detailed analysis
4. **Review regularly** and track trends
5. **Automate monitoring** and alerting

---

## Further Reading

- MySQL Slow Query Log documentation
- Percona Toolkit pt-query-digest manual
- "High Performance MySQL" - Query analysis
