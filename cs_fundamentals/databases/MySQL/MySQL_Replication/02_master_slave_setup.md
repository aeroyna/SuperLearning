# Master-Slave Replication Setup

## Learning Objectives
- Set up source-replica replication from scratch
- Configure replication users and security
- Handle initial data synchronization
- Manage replication topology

---

## 1. Prerequisites

### Server Requirements

```
Source (Master):
- Unique server_id
- Binary logging enabled
- Replication user created

Replica (Slave):
- Unique server_id
- Sufficient disk space for relay logs
- Network connectivity to source
```

### Network Configuration

```bash
# Ensure connectivity
# From replica to source
mysql -h source.example.com -P 3306 -u repl -p

# Firewall rules (if needed)
# Allow port 3306 from replica to source
```

---

## 2. Source (Master) Configuration

### Configuration File

```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf (or my.cnf)

[mysqld]
# Unique server ID (1-2^32-1)
server_id = 1

# Enable binary logging
log_bin = /var/lib/mysql/binlog
binlog_format = ROW

# GTID mode (recommended)
gtid_mode = ON
enforce_gtid_consistency = ON

# Binary log retention
binlog_expire_logs_seconds = 604800

# Sync for durability
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

# Allow replicas to connect
bind-address = 0.0.0.0
```

### Create Replication User

```sql
-- Create dedicated replication user
CREATE USER 'repl'@'%' IDENTIFIED BY 'secure_password_here';

-- Grant replication privileges
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

-- For GTID-based replication (MySQL 8.0)
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'repl'@'%';

-- Flush privileges
FLUSH PRIVILEGES;

-- Verify
SHOW GRANTS FOR 'repl'@'%';
```

### Restart and Verify

```bash
# Restart MySQL
sudo systemctl restart mysql

# Verify settings
mysql -e "SHOW VARIABLES LIKE 'server_id'"
mysql -e "SHOW VARIABLES LIKE 'log_bin'"
mysql -e "SHOW MASTER STATUS"
```

---

## 3. Initial Data Transfer

### Option 1: mysqldump (Small Databases)

```bash
# On source: Create consistent backup
mysqldump --all-databases \
          --source-data=2 \
          --single-transaction \
          --routines \
          --triggers \
          --events \
          --set-gtid-purged=ON \
          -u root -p > backup.sql

# Transfer to replica
scp backup.sql replica.example.com:/tmp/

# On replica: Import
mysql -u root -p < /tmp/backup.sql
```

### Option 2: mysqlpump (Parallel Dump)

```bash
# Faster for large databases
mysqlpump --all-databases \
          --add-drop-database \
          --single-transaction \
          --set-gtid-purged=ON \
          --default-parallelism=4 \
          -u root -p > backup.sql
```

### Option 3: Percona XtraBackup (Large Databases)

```bash
# On source: Create hot backup
xtrabackup --backup \
           --target-dir=/backup/base \
           --user=root \
           --password=xxx

# Prepare backup
xtrabackup --prepare --target-dir=/backup/base

# Transfer to replica
rsync -avz /backup/base/ replica.example.com:/var/lib/mysql/

# On replica: Set permissions
chown -R mysql:mysql /var/lib/mysql/
```

### Option 4: Clone Plugin (MySQL 8.0)

```sql
-- On replica: Use clone plugin for fast provisioning
INSTALL PLUGIN clone SONAME 'mysql_clone.so';

-- Clone from source
CLONE INSTANCE FROM 'repl'@'source.example.com':3306
IDENTIFIED BY 'secure_password_here';

-- Replica restarts automatically after clone
```

---

## 4. Replica (Slave) Configuration

### Configuration File

```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
# Unique server ID (different from source)
server_id = 2

# Relay log settings
relay_log = /var/lib/mysql/relay-bin
relay_log_recovery = ON

# GTID mode (must match source)
gtid_mode = ON
enforce_gtid_consistency = ON
log_slave_updates = ON

# Read-only mode (recommended for replicas)
read_only = ON
super_read_only = ON  # Prevents even SUPER users from writing

# Binary logging on replica (for chained replication)
log_bin = /var/lib/mysql/binlog
```

### Configure Replication

```sql
-- MySQL 8.0.22+ syntax
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'source.example.com',
    SOURCE_PORT = 3306,
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'secure_password_here',
    SOURCE_AUTO_POSITION = 1;  -- GTID mode

-- Legacy syntax (pre-8.0.22)
CHANGE MASTER TO
    MASTER_HOST = 'source.example.com',
    MASTER_PORT = 3306,
    MASTER_USER = 'repl',
    MASTER_PASSWORD = 'secure_password_here',
    MASTER_AUTO_POSITION = 1;
```

### Non-GTID Setup (Position-Based)

```sql
-- If not using GTID, get position from source
-- SHOW MASTER STATUS on source, note File and Position

CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'source.example.com',
    SOURCE_PORT = 3306,
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'secure_password_here',
    SOURCE_LOG_FILE = 'binlog.000003',
    SOURCE_LOG_POS = 897;
```

### Start Replication

```sql
-- Start replication
START REPLICA;

-- Legacy
START SLAVE;

-- Verify status
SHOW REPLICA STATUS\G
```

---

## 5. Verify Replication

### Check Replica Status

```sql
SHOW REPLICA STATUS\G

-- Key fields to check:
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: source.example.com
                  Source_User: repl
                  Source_Port: 3306
            Replica_IO_Running: Yes      -- Should be Yes
           Replica_SQL_Running: Yes      -- Should be Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
        Seconds_Behind_Source: 0         -- Should be 0 or low
                Last_IO_Error:           -- Should be empty
               Last_SQL_Error:           -- Should be empty
           Retrieved_Gtid_Set: 3E11FA47-71CA-11E1-9E33:1-150
            Executed_Gtid_Set: 3E11FA47-71CA-11E1-9E33:1-150
```

### Test Replication

```sql
-- On source: Create test data
USE test;
CREATE TABLE repl_test (id INT PRIMARY KEY, msg VARCHAR(100));
INSERT INTO repl_test VALUES (1, 'Replication working!');

-- On replica: Verify data appears
USE test;
SELECT * FROM repl_test;
```

---

## 6. Semi-Synchronous Replication

### Enable Semi-Sync

```sql
-- On source
INSTALL PLUGIN rpl_semi_sync_source SONAME 'semisync_source.so';
SET GLOBAL rpl_semi_sync_source_enabled = ON;
SET GLOBAL rpl_semi_sync_source_timeout = 10000;  -- 10 seconds

-- On replica
INSTALL PLUGIN rpl_semi_sync_replica SONAME 'semisync_replica.so';
SET GLOBAL rpl_semi_sync_replica_enabled = ON;

-- Restart replication
STOP REPLICA;
START REPLICA;
```

### Semi-Sync Configuration

```ini
# my.cnf on source
[mysqld]
plugin_load_add = semisync_source.so
rpl_semi_sync_source_enabled = ON
rpl_semi_sync_source_timeout = 10000

# my.cnf on replica
[mysqld]
plugin_load_add = semisync_replica.so
rpl_semi_sync_replica_enabled = ON
```

### Monitor Semi-Sync

```sql
-- On source
SHOW STATUS LIKE 'Rpl_semi_sync%';

-- Key metrics:
-- Rpl_semi_sync_source_status: ON
-- Rpl_semi_sync_source_clients: 1 (number of semi-sync replicas)
-- Rpl_semi_sync_source_yes_tx: 1000 (transactions with ack)
-- Rpl_semi_sync_source_no_tx: 5 (transactions without ack - timeout)
```

---

## 7. Multi-Source Replication

### Configure Multiple Sources

```sql
-- MySQL 5.7+ supports multiple replication channels

-- Add first source
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'source1.example.com',
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'password',
    SOURCE_AUTO_POSITION = 1
FOR CHANNEL 'source1';

-- Add second source
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'source2.example.com',
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'password',
    SOURCE_AUTO_POSITION = 1
FOR CHANNEL 'source2';

-- Start both channels
START REPLICA FOR CHANNEL 'source1';
START REPLICA FOR CHANNEL 'source2';

-- Check status for each
SHOW REPLICA STATUS FOR CHANNEL 'source1'\G
SHOW REPLICA STATUS FOR CHANNEL 'source2'\G
```

---

## 8. Replication Topology Patterns

### Chain Replication

```
Source → Replica1 → Replica2 → Replica3

-- Replica1 config: log_slave_updates = ON
-- Each replica replicates from previous
-- Reduces load on source
```

### Star Topology

```
           Source
          /  |  \
    Replica1 Replica2 Replica3

-- All replicas connect directly to source
-- Simple management
-- Higher source network load
```

### Tree Topology

```
           Source
          /      \
    Replica1    Replica2
      /  \          |
   Rep3  Rep4    Rep5

-- Hierarchical structure
-- Reduces source connections
-- Intermediate replicas need log_slave_updates = ON
```

---

## 9. Failover Procedures

### Promote Replica to Source

```sql
-- On replica being promoted:

-- 1. Stop replication
STOP REPLICA;

-- 2. Ensure all relay logs are applied
-- Wait for Exec_Master_Log_Pos = Read_Master_Log_Pos

-- 3. Reset replica configuration
RESET REPLICA ALL;

-- 4. Enable writes
SET GLOBAL read_only = OFF;
SET GLOBAL super_read_only = OFF;

-- 5. Get binary log position
SHOW MASTER STATUS;

-- 6. Point other replicas to new source
-- On other replicas:
STOP REPLICA;
CHANGE REPLICATION SOURCE TO
    SOURCE_HOST = 'new_source.example.com',
    SOURCE_AUTO_POSITION = 1;  -- GTID makes this easy
START REPLICA;
```

### Automated Failover (MySQL Router)

```ini
# mysqlrouter.conf
[routing:primary]
bind_address = 0.0.0.0
bind_port = 6446
destinations = source:3306,replica1:3306,replica2:3306
routing_strategy = first-available
protocol = classic
```

---

## Summary

| Step | Action |
|------|--------|
| 1 | Configure source with binary logging |
| 2 | Create replication user |
| 3 | Transfer initial data |
| 4 | Configure replica |
| 5 | Start and verify replication |
| 6 | Optional: Enable semi-sync |

---

## Further Reading

- MySQL Replication documentation
- "High Availability MySQL Cookbook"
- Percona blog on replication best practices
