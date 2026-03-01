# Point-in-Time Recovery

## What is Point-in-Time Recovery?

```
┌─────────────────────────────────────────────────────────────────┐
│              Point-in-Time Recovery (PITR)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONCEPT                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Restore database to any specific moment in time           │ │
│  │                                                             │ │
│  │ Base Backup           Continuous Changes          Target   │ │
│  │     │                      │                        │      │ │
│  │     ▼                      ▼                        ▼      │ │
│  │ ────●──────────────────────────────────────────────●────   │ │
│  │     │                                               │      │ │
│  │     └─────────── WAL/Binlog Applied ───────────────┘      │ │
│  │                                                             │ │
│  │ Use Cases:                                                 │ │
│  │ • Recover from accidental DELETE                          │ │
│  │ • Undo bad UPDATE statement                               │ │
│  │ • Restore to moment before corruption                     │ │
│  │ • Recover from ransomware (restore pre-attack)            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMPONENTS                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │ │
│  │  │    Base     │ +  │ Transaction │ =  │   Restored  │    │ │
│  │  │   Backup    │    │    Logs     │    │   Database  │    │ │
│  │  └─────────────┘    └─────────────┘    └─────────────┘    │ │
│  │                                                             │ │
│  │  PostgreSQL: pg_basebackup + WAL files                    │ │
│  │  MySQL: xtrabackup + Binary logs                          │ │
│  │  SQL Server: Full backup + Transaction logs               │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## PostgreSQL PITR

```
┌─────────────────────────────────────────────────────────────────┐
│              PostgreSQL Point-in-Time Recovery                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: CONFIGURE WAL ARCHIVING                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # postgresql.conf                                          │ │
│  │                                                             │ │
│  │ # Enable WAL archiving                                     │ │
│  │ wal_level = replica                                        │ │
│  │ archive_mode = on                                          │ │
│  │ archive_command = 'cp %p /archive/wal/%f'                  │ │
│  │                                                             │ │
│  │ # Alternative: Archive to S3                               │ │
│  │ archive_command = 'aws s3 cp %p s3://bucket/wal/%f'       │ │
│  │                                                             │ │
│  │ # Ensure continuous archiving                              │ │
│  │ archive_timeout = 300  # Force switch every 5 minutes     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 2: TAKE BASE BACKUP                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Create base backup                                       │ │
│  │ pg_basebackup -h localhost -U replicator \                │ │
│  │   -D /backup/base -Fp -Xs -P                              │ │
│  │                                                             │ │
│  │ # Options explained:                                       │ │
│  │ # -Fp: Plain format (file directory)                      │ │
│  │ # -Xs: Stream WAL during backup                           │ │
│  │ # -P : Show progress                                       │ │
│  │                                                             │ │
│  │ # Record the backup label                                  │ │
│  │ cat /backup/base/backup_label                             │ │
│  │ # START WAL LOCATION: 0/5000028                           │ │
│  │ # START TIME: 2024-01-15 10:30:00 UTC                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 3: RESTORE TO POINT IN TIME                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Stop PostgreSQL                                          │ │
│  │ systemctl stop postgresql                                  │ │
│  │                                                             │ │
│  │ # Clear existing data directory                           │ │
│  │ rm -rf /var/lib/postgresql/14/main/*                      │ │
│  │                                                             │ │
│  │ # Restore base backup                                      │ │
│  │ cp -r /backup/base/* /var/lib/postgresql/14/main/         │ │
│  │                                                             │ │
│  │ # Create recovery configuration                           │ │
│  │ cat > /var/lib/postgresql/14/main/postgresql.auto.conf << EOF │
│  │ restore_command = 'cp /archive/wal/%f %p'                 │ │
│  │ recovery_target_time = '2024-01-15 14:30:00 UTC'          │ │
│  │ recovery_target_action = 'promote'                        │ │
│  │ EOF                                                        │ │
│  │                                                             │ │
│  │ # Create recovery signal file                             │ │
│  │ touch /var/lib/postgresql/14/main/recovery.signal         │ │
│  │                                                             │ │
│  │ # Start PostgreSQL                                         │ │
│  │ systemctl start postgresql                                 │ │
│  │                                                             │ │
│  │ # Check recovery status                                    │ │
│  │ psql -c "SELECT pg_is_in_recovery();"                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RECOVERY TARGET OPTIONS                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # By timestamp                                             │ │
│  │ recovery_target_time = '2024-01-15 14:30:00 UTC'          │ │
│  │                                                             │ │
│  │ # By transaction ID                                        │ │
│  │ recovery_target_xid = '12345678'                          │ │
│  │                                                             │ │
│  │ # By named restore point                                   │ │
│  │ SELECT pg_create_restore_point('before_migration');       │ │
│  │ recovery_target_name = 'before_migration'                 │ │
│  │                                                             │ │
│  │ # By LSN (Log Sequence Number)                            │ │
│  │ recovery_target_lsn = '0/6000000'                         │ │
│  │                                                             │ │
│  │ # Inclusive/Exclusive                                      │ │
│  │ recovery_target_inclusive = false  # Stop before target   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## MySQL PITR

```
┌─────────────────────────────────────────────────────────────────┐
│              MySQL Point-in-Time Recovery                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: CONFIGURE BINARY LOGGING                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # my.cnf                                                   │ │
│  │ [mysqld]                                                   │ │
│  │ server-id = 1                                              │ │
│  │ log_bin = /var/log/mysql/mysql-bin                        │ │
│  │ binlog_format = ROW                                        │ │
│  │ expire_logs_days = 14                                      │ │
│  │ sync_binlog = 1                                            │ │
│  │                                                             │ │
│  │ # Verify binary logging                                    │ │
│  │ mysql> SHOW VARIABLES LIKE 'log_bin';                     │ │
│  │ +---------------+-------+                                  │ │
│  │ | Variable_name | Value |                                  │ │
│  │ +---------------+-------+                                  │ │
│  │ | log_bin       | ON    |                                  │ │
│  │ +---------------+-------+                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 2: TAKE BACKUP WITH BINLOG POSITION                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Using mysqldump                                          │ │
│  │ mysqldump --single-transaction --master-data=2 \          │ │
│  │   --all-databases > backup.sql                             │ │
│  │                                                             │ │
│  │ # Check binlog position in backup                         │ │
│  │ head -30 backup.sql | grep "CHANGE MASTER"                │ │
│  │ # -- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000005', │ │
│  │ # -- MASTER_LOG_POS=154;                                  │ │
│  │                                                             │ │
│  │ # Using xtrabackup                                         │ │
│  │ xtrabackup --backup --target-dir=/backup/full             │ │
│  │ cat /backup/full/xtrabackup_binlog_info                   │ │
│  │ # mysql-bin.000005  154                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 3: IDENTIFY RECOVERY POINT                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # List binary log files                                    │ │
│  │ mysql> SHOW BINARY LOGS;                                   │ │
│  │                                                             │ │
│  │ # Find the bad statement                                   │ │
│  │ mysqlbinlog --start-datetime="2024-01-15 14:00:00" \      │ │
│  │   --stop-datetime="2024-01-15 15:00:00" \                 │ │
│  │   /var/log/mysql/mysql-bin.000005 | less                  │ │
│  │                                                             │ │
│  │ # Find position of bad statement                          │ │
│  │ # at 12345                                                │ │
│  │ # DROP TABLE users                                        │ │
│  │                                                             │ │
│  │ # Stop position should be just before the bad statement   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 4: RESTORE                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Restore full backup                                      │ │
│  │ mysql < backup.sql                                         │ │
│  │                                                             │ │
│  │ # Apply binary logs up to the point before the error     │ │
│  │ mysqlbinlog --stop-position=12344 \                       │ │
│  │   /var/log/mysql/mysql-bin.000005 \                       │ │
│  │   /var/log/mysql/mysql-bin.000006 | mysql                 │ │
│  │                                                             │ │
│  │ # Or by timestamp                                          │ │
│  │ mysqlbinlog --stop-datetime="2024-01-15 14:29:59" \       │ │
│  │   /var/log/mysql/mysql-bin.000005 | mysql                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SKIP SPECIFIC TRANSACTION                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Apply logs, skipping the bad statement                  │ │
│  │ mysqlbinlog --start-position=154 \                        │ │
│  │   --stop-position=12344 \                                 │ │
│  │   /var/log/mysql/mysql-bin.000005 | mysql                 │ │
│  │                                                             │ │
│  │ # Skip the bad statement, continue from after             │ │
│  │ mysqlbinlog --start-position=12500 \                      │ │
│  │   /var/log/mysql/mysql-bin.000005 \                       │ │
│  │   /var/log/mysql/mysql-bin.000006 | mysql                 │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Using pgBackRest for PITR

```
┌─────────────────────────────────────────────────────────────────┐
│              pgBackRest Point-in-Time Recovery                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONFIGURATION                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # /etc/pgbackrest/pgbackrest.conf                         │ │
│  │ [global]                                                   │ │
│  │ repo1-path=/backup/pgbackrest                             │ │
│  │ repo1-retention-full=2                                     │ │
│  │ repo1-retention-diff=7                                     │ │
│  │ process-max=4                                              │ │
│  │ compress-type=zst                                          │ │
│  │                                                             │ │
│  │ [main]                                                     │ │
│  │ pg1-path=/var/lib/postgresql/14/main                      │ │
│  │                                                             │ │
│  │ # postgresql.conf                                          │ │
│  │ archive_mode = on                                          │ │
│  │ archive_command = 'pgbackrest --stanza=main archive-push %p' │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CREATE STANZA AND BACKUP                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Create stanza                                            │ │
│  │ pgbackrest --stanza=main stanza-create                    │ │
│  │                                                             │ │
│  │ # Full backup                                              │ │
│  │ pgbackrest --stanza=main --type=full backup               │ │
│  │                                                             │ │
│  │ # Incremental backup                                       │ │
│  │ pgbackrest --stanza=main --type=incr backup               │ │
│  │                                                             │ │
│  │ # List backups                                             │ │
│  │ pgbackrest --stanza=main info                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POINT-IN-TIME RESTORE                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Stop PostgreSQL                                          │ │
│  │ systemctl stop postgresql                                  │ │
│  │                                                             │ │
│  │ # Restore to specific time                                 │ │
│  │ pgbackrest --stanza=main --delta \                        │ │
│  │   --type=time \                                            │ │
│  │   --target="2024-01-15 14:30:00" \                        │ │
│  │   --target-action=promote \                                │ │
│  │   restore                                                  │ │
│  │                                                             │ │
│  │ # Start PostgreSQL                                         │ │
│  │ systemctl start postgresql                                 │ │
│  │                                                             │ │
│  │ # Other restore options:                                   │ │
│  │ --type=xid --target="12345"     # By transaction ID       │ │
│  │ --type=name --target="point1"   # By restore point        │ │
│  │ --type=lsn --target="0/6000000" # By LSN                  │ │
│  │ --type=immediate                 # Restore and stop       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DELTA RESTORE                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Delta restore only copies changed files                 │ │
│  │ # Much faster for large databases                         │ │
│  │                                                             │ │
│  │ # Regular restore: Copies ALL files                       │ │
│  │ pgbackrest --stanza=main restore                          │ │
│  │                                                             │ │
│  │ # Delta restore: Only copies changed files                │ │
│  │ pgbackrest --stanza=main --delta restore                  │ │
│  │                                                             │ │
│  │ Comparison (500GB database):                               │ │
│  │ • Regular: ~2 hours (copy all files)                      │ │
│  │ • Delta: ~10 minutes (only changed files)                 │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Flashback and Temporal Queries

```
┌─────────────────────────────────────────────────────────────────┐
│              Alternative Recovery Methods                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ORACLE FLASHBACK                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Flashback query (see past data)                        │ │
│  │ SELECT * FROM users AS OF TIMESTAMP                       │ │
│  │   TO_TIMESTAMP('2024-01-15 14:00:00', 'YYYY-MM-DD HH24:MI:SS'); │
│  │                                                             │ │
│  │ -- Flashback table (restore table to past)               │ │
│  │ FLASHBACK TABLE users TO TIMESTAMP                        │ │
│  │   TO_TIMESTAMP('2024-01-15 14:00:00', 'YYYY-MM-DD HH24:MI:SS'); │
│  │                                                             │ │
│  │ -- Flashback database (entire database)                   │ │
│  │ SHUTDOWN IMMEDIATE;                                        │ │
│  │ STARTUP MOUNT;                                             │ │
│  │ FLASHBACK DATABASE TO TIMESTAMP                           │ │
│  │   TO_TIMESTAMP('2024-01-15 14:00:00', 'YYYY-MM-DD HH24:MI:SS'); │
│  │ ALTER DATABASE OPEN RESETLOGS;                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SQL SERVER TEMPORAL TABLES                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Create system-versioned table                          │ │
│  │ CREATE TABLE users (                                       │ │
│  │   id INT PRIMARY KEY,                                     │ │
│  │   name VARCHAR(100),                                       │ │
│  │   email VARCHAR(255),                                      │ │
│  │   SysStartTime DATETIME2 GENERATED ALWAYS AS ROW START,  │ │
│  │   SysEndTime DATETIME2 GENERATED ALWAYS AS ROW END,      │ │
│  │   PERIOD FOR SYSTEM_TIME (SysStartTime, SysEndTime)       │ │
│  │ ) WITH (SYSTEM_VERSIONING = ON);                          │ │
│  │                                                             │ │
│  │ -- Query historical data                                   │ │
│  │ SELECT * FROM users                                        │ │
│  │ FOR SYSTEM_TIME AS OF '2024-01-15 14:00:00';              │ │
│  │                                                             │ │
│  │ -- Query data between two points                          │ │
│  │ SELECT * FROM users                                        │ │
│  │ FOR SYSTEM_TIME BETWEEN '2024-01-15 14:00:00'            │ │
│  │   AND '2024-01-15 15:00:00';                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL TEMPORAL APPROACH                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Using pg_temporal extension or manual implementation   │ │
│  │                                                             │ │
│  │ CREATE TABLE users_history (                               │ │
│  │   id INT,                                                  │ │
│  │   name VARCHAR(100),                                       │ │
│  │   email VARCHAR(255),                                      │ │
│  │   valid_from TIMESTAMPTZ NOT NULL,                        │ │
│  │   valid_to TIMESTAMPTZ NOT NULL DEFAULT 'infinity',       │ │
│  │   PRIMARY KEY (id, valid_from)                            │ │
│  │ );                                                         │ │
│  │                                                             │ │
│  │ -- Query data as of specific time                         │ │
│  │ SELECT * FROM users_history                                │ │
│  │ WHERE valid_from <= '2024-01-15 14:00:00'                 │ │
│  │   AND valid_to > '2024-01-15 14:00:00';                   │ │
│  │                                                             │ │
│  │ -- Trigger to maintain history                             │ │
│  │ CREATE OR REPLACE FUNCTION track_user_changes()           │ │
│  │ RETURNS TRIGGER AS $$                                      │ │
│  │ BEGIN                                                      │ │
│  │   IF TG_OP = 'UPDATE' THEN                                │ │
│  │     UPDATE users_history SET valid_to = NOW()             │ │
│  │     WHERE id = OLD.id AND valid_to = 'infinity';          │ │
│  │     INSERT INTO users_history (id, name, email, valid_from) │
│  │     VALUES (NEW.id, NEW.name, NEW.email, NOW());          │ │
│  │   END IF;                                                  │ │
│  │   RETURN NEW;                                              │ │
│  │ END;                                                       │ │
│  │ $$ LANGUAGE plpgsql;                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## PITR Best Practices

```
┌─────────────────────────────────────────────────────────────────┐
│              PITR Best Practices                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WAL/BINLOG MANAGEMENT                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Archive continuously, not just during backups           │ │
│  │ • Monitor archive lag                                      │ │
│  │ • Keep logs longer than backup retention                  │ │
│  │ • Use compression for archived logs                       │ │
│  │ • Replicate logs to offsite storage                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RECOVERY PREPARATION                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Document recovery procedures                             │ │
│  │ • Practice recovery regularly                              │ │
│  │ • Know how to find transaction timestamps                 │ │
│  │ • Keep recovery tools accessible                          │ │
│  │ • Test recovery in isolated environment first            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MONITORING                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # PostgreSQL: Monitor archive status                      │ │
│  │ SELECT * FROM pg_stat_archiver;                           │ │
│  │                                                             │ │
│  │ # Check last archived WAL                                  │ │
│  │ SELECT last_archived_wal, last_archived_time              │ │
│  │ FROM pg_stat_archiver;                                     │ │
│  │                                                             │ │
│  │ # MySQL: Monitor binlog status                             │ │
│  │ SHOW MASTER STATUS;                                        │ │
│  │ SHOW BINARY LOGS;                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMMON PITFALLS                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✗ Deleting WAL files before backup expires               │ │
│  │ ✗ Not testing recovery procedures                         │ │
│  │ ✗ Missing archive gaps                                    │ │
│  │ ✗ Wrong timezone in recovery target                       │ │
│  │ ✗ Insufficient disk space for archived logs              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
