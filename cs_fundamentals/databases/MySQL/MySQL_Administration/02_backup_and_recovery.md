# Backup and Recovery

## Learning Objectives
- Understand backup types and strategies
- Master mysqldump and physical backup tools
- Implement point-in-time recovery
- Design backup policies for different scenarios

---

## 1. Backup Types

### Logical vs Physical Backups

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Backup Type Comparison                            │
│                                                                      │
│  LOGICAL BACKUP (mysqldump)                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • SQL statements (CREATE TABLE, INSERT)                    │    │
│  │  • Portable across MySQL versions                           │    │
│  │  • Slower backup and restore                                │    │
│  │  • Can backup specific tables                               │    │
│  │  • Larger backup size (text format)                         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  PHYSICAL BACKUP (file copy, xtrabackup)                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Raw data files                                           │    │
│  │  • Fast backup and restore                                  │    │
│  │  • Version-specific                                         │    │
│  │  • Full database only (usually)                             │    │
│  │  • Smaller backup size (binary)                             │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Full vs Incremental

```
Full Backup:       [████████████████████] 100%
                          Day 1

Incremental:       [████████████████████] Full (Day 1)
                   [███]                  Inc 1 (Day 2)
                   [█████]                Inc 2 (Day 3)
                   [██]                   Inc 3 (Day 4)
```

---

## 2. mysqldump

### Basic Usage

```bash
# Single database
mysqldump -u root -p mydb > mydb_backup.sql

# Multiple databases
mysqldump -u root -p --databases db1 db2 db3 > multiple_dbs.sql

# All databases
mysqldump -u root -p --all-databases > full_backup.sql

# Specific tables
mysqldump -u root -p mydb table1 table2 > tables_backup.sql
```

### Production Options

```bash
# Recommended production backup
mysqldump -u root -p \
    --single-transaction \          # Consistent snapshot (InnoDB)
    --routines \                    # Include stored procedures
    --triggers \                    # Include triggers
    --events \                      # Include events
    --set-gtid-purged=ON \          # GTID info for replication
    --source-data=2 \               # Binary log position (commented)
    --flush-logs \                  # Rotate binary logs
    --max-allowed-packet=512M \     # Large packets
    mydb > mydb_backup.sql

# Compress output
mysqldump -u root -p mydb | gzip > mydb_backup.sql.gz

# With parallel compression
mysqldump -u root -p mydb | pigz > mydb_backup.sql.gz
```

### Structure Only

```bash
# Schema only (no data)
mysqldump -u root -p --no-data mydb > schema.sql

# Data only (no schema)
mysqldump -u root -p --no-create-info mydb > data.sql
```

### mysqldump Options Reference

| Option | Description |
|--------|-------------|
| --single-transaction | Consistent backup for InnoDB |
| --lock-tables | Lock MyISAM tables |
| --routines | Include procedures/functions |
| --triggers | Include triggers |
| --events | Include events |
| --source-data | Include binlog position |
| --set-gtid-purged | Include GTID information |
| --where | Filter rows with condition |

---

## 3. Physical Backups

### Percona XtraBackup

```bash
# Install
apt-get install percona-xtrabackup-80

# Full backup
xtrabackup --backup \
    --target-dir=/backup/full \
    --user=root --password=xxx

# Prepare backup (apply logs)
xtrabackup --prepare --target-dir=/backup/full

# Incremental backup
xtrabackup --backup \
    --target-dir=/backup/inc1 \
    --incremental-basedir=/backup/full \
    --user=root --password=xxx

# Prepare incremental
xtrabackup --prepare --apply-log-only --target-dir=/backup/full
xtrabackup --prepare --target-dir=/backup/full \
    --incremental-dir=/backup/inc1
```

### MySQL Enterprise Backup

```bash
# Hot backup
mysqlbackup --user=root --password=xxx \
    --backup-dir=/backup/full backup

# Incremental
mysqlbackup --user=root --password=xxx \
    --backup-dir=/backup/inc1 \
    --incremental \
    --incremental-base=dir:/backup/full \
    backup-and-apply-log
```

### Clone Plugin (MySQL 8.0)

```sql
-- Install plugin
INSTALL PLUGIN clone SONAME 'mysql_clone.so';

-- Local clone
CLONE LOCAL DATA DIRECTORY = '/backup/clone';

-- Remote clone
CLONE INSTANCE FROM 'user'@'source_host':3306
    IDENTIFIED BY 'password';
```

---

## 4. Restoration

### Restore from mysqldump

```bash
# Basic restore
mysql -u root -p mydb < mydb_backup.sql

# Restore compressed backup
gunzip < mydb_backup.sql.gz | mysql -u root -p mydb

# Restore all databases
mysql -u root -p < full_backup.sql

# With progress indicator
pv mydb_backup.sql | mysql -u root -p mydb
```

### Restore from Physical Backup

```bash
# Stop MySQL
systemctl stop mysql

# Remove existing data
rm -rf /var/lib/mysql/*

# Copy backup files
xtrabackup --copy-back --target-dir=/backup/full

# Fix permissions
chown -R mysql:mysql /var/lib/mysql

# Start MySQL
systemctl start mysql
```

### Partial Restore

```sql
-- Restore single table from backup
-- 1. Restore backup to temporary location
-- 2. Extract table data

-- From mysqldump: Extract CREATE TABLE and INSERT statements
grep -A 1000 "CREATE TABLE \`users\`" backup.sql | \
    grep -B 1000 "UNLOCK TABLES" > users_only.sql

-- Import
mysql -u root -p mydb < users_only.sql
```

---

## 5. Point-in-Time Recovery (PITR)

### Recovery Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Point-in-Time Recovery                            │
│                                                                      │
│  Full Backup               Binary Logs           Target Point       │
│  (Day 1 00:00)            (ongoing)              (Day 3 14:30)      │
│       │                        │                      │             │
│       ▼                        ▼                      ▼             │
│  [████████] + [binlog.001][binlog.002][binlog.003] = [████████████] │
│                                                                      │
│  Steps:                                                              │
│  1. Restore full backup                                              │
│  2. Apply binary logs up to target time                              │
└─────────────────────────────────────────────────────────────────────┘
```

### PITR Steps

```bash
# 1. Restore base backup
mysql -u root -p < full_backup.sql

# 2. Find binary log position from backup
grep "CHANGE MASTER" full_backup.sql
# -- CHANGE MASTER TO MASTER_LOG_FILE='binlog.000003', MASTER_LOG_POS=154;

# 3. Apply binary logs up to recovery point
mysqlbinlog \
    --start-position=154 \
    --stop-datetime="2024-01-15 14:30:00" \
    binlog.000003 binlog.000004 | mysql -u root -p

# With GTID
mysqlbinlog \
    --include-gtids="server-uuid:1-1000" \
    binlog.000003 binlog.000004 | mysql -u root -p
```

### Skip Problematic Transaction

```bash
# Find problematic statement
mysqlbinlog binlog.000004 | grep -A 5 "DROP TABLE"

# Skip specific GTID
mysqlbinlog \
    --exclude-gtids="server-uuid:500" \
    binlog.000004 | mysql -u root -p
```

---

## 6. Backup Strategies

### Small Database (< 10GB)

```bash
#!/bin/bash
# daily_backup.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR=/backup/mysql
RETENTION_DAYS=7

# Full logical backup daily
mysqldump -u root -p'password' \
    --all-databases \
    --single-transaction \
    --routines --triggers --events \
    --set-gtid-purged=ON | \
    gzip > ${BACKUP_DIR}/full_${DATE}.sql.gz

# Cleanup old backups
find ${BACKUP_DIR} -name "*.sql.gz" -mtime +${RETENTION_DAYS} -delete
```

### Medium Database (10-100GB)

```bash
#!/bin/bash
# Weekly full, daily incremental

DAY=$(date +%u)  # 1-7 (Mon-Sun)
DATE=$(date +%Y%m%d)
BACKUP_DIR=/backup/mysql

if [ "$DAY" -eq 1 ]; then
    # Monday: Full backup
    xtrabackup --backup \
        --target-dir=${BACKUP_DIR}/full_${DATE} \
        --user=backup --password=xxx
    xtrabackup --prepare --target-dir=${BACKUP_DIR}/full_${DATE}
else
    # Other days: Incremental
    LAST_FULL=$(ls -d ${BACKUP_DIR}/full_* | tail -1)
    xtrabackup --backup \
        --target-dir=${BACKUP_DIR}/inc_${DATE} \
        --incremental-basedir=${LAST_FULL} \
        --user=backup --password=xxx
fi
```

### Large Database (> 100GB)

```bash
# Streaming backup to remote storage
xtrabackup --backup \
    --stream=xbstream \
    --parallel=4 \
    --compress \
    --compress-threads=4 \
    --user=backup --password=xxx | \
    ssh backup@storage-server "cat > /backup/mysql_$(date +%Y%m%d).xbstream"
```

---

## 7. Backup Verification

### Verify Backup Integrity

```bash
# Check mysqldump file
gzip -t backup.sql.gz
echo $?  # 0 = OK

# Check SQL syntax
mysql -u root -p --execute="SOURCE backup.sql" --database=test_restore

# Count objects
grep -c "CREATE TABLE" backup.sql
grep -c "INSERT INTO" backup.sql
```

### Test Restore

```bash
# Restore to test instance
mysql -h test-server -u root -p test_db < backup.sql

# Verify row counts
mysql -e "SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema='test_db'"

# Compare with production
pt-table-checksum h=production,D=mydb h=test,D=mydb
```

### Automated Verification

```bash
#!/bin/bash
# verify_backup.sh

BACKUP_FILE=$1
TEST_DB="restore_test_$(date +%s)"

# Create test database
mysql -e "CREATE DATABASE ${TEST_DB}"

# Restore
gunzip < ${BACKUP_FILE} | mysql ${TEST_DB}

# Verify table count
TABLE_COUNT=$(mysql -N -e "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='${TEST_DB}'")

if [ "$TABLE_COUNT" -gt 0 ]; then
    echo "Backup verified: ${TABLE_COUNT} tables restored"
else
    echo "ERROR: No tables restored"
    exit 1
fi

# Cleanup
mysql -e "DROP DATABASE ${TEST_DB}"
```

---

## 8. Backup Best Practices

### Security

```bash
# Encrypt backup
mysqldump ... | openssl enc -aes-256-cbc -salt -pass file:/path/to/key > backup.sql.enc

# Decrypt
openssl enc -d -aes-256-cbc -pass file:/path/to/key < backup.sql.enc | mysql

# Secure backup user
CREATE USER 'backup'@'localhost' IDENTIFIED BY 'StrongPass!';
GRANT SELECT, LOCK TABLES, SHOW VIEW, EVENT, TRIGGER,
      RELOAD, REPLICATION CLIENT ON *.* TO 'backup'@'localhost';
```

### Monitoring

```sql
-- Create backup log table
CREATE TABLE backup_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    backup_type ENUM('full', 'incremental'),
    start_time DATETIME,
    end_time DATETIME,
    status ENUM('success', 'failed'),
    size_bytes BIGINT,
    file_path VARCHAR(500),
    error_message TEXT
);

-- Log backup completion
INSERT INTO backup_log (backup_type, start_time, end_time, status, size_bytes, file_path)
VALUES ('full', @start, NOW(), 'success', @size, '/backup/full_20240115.sql.gz');
```

---

## Summary

| Method | Speed | Size | Use Case |
|--------|-------|------|----------|
| mysqldump | Slow | Large | Small DBs, portability |
| xtrabackup | Fast | Small | Large DBs, production |
| Clone plugin | Fast | Small | MySQL 8.0, provisioning |

### Backup Checklist

1. [ ] Daily backup automated
2. [ ] Offsite copy maintained
3. [ ] Encryption enabled
4. [ ] Weekly restore test
5. [ ] Binary logs retained for PITR
6. [ ] Monitoring and alerting

---

## Further Reading

- MySQL Backup and Recovery documentation
- Percona XtraBackup documentation
- "High Availability MySQL" - Backup strategies
