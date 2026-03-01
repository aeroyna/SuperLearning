# Backup and Recovery

## Learning Objectives
- Understand PostgreSQL backup strategies
- Use pg_dump and pg_basebackup effectively
- Implement point-in-time recovery (PITR)
- Configure automated backup tools

---

## 1. Backup Strategies

### Backup Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Backup Types                           │
│                                                                      │
│  LOGICAL BACKUP (pg_dump):                                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • SQL statements to recreate database                      │    │
│  │  • Portable across versions                                 │    │
│  │  • Selective (tables, schemas)                              │    │
│  │  • Slower for large databases                               │    │
│  │  • No PITR capability                                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  PHYSICAL BACKUP (pg_basebackup):                                    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Binary copy of data files                                │    │
│  │  • Fast for large databases                                 │    │
│  │  • Same PostgreSQL version required                         │    │
│  │  • PITR capable (with WAL archiving)                         │    │
│  │  • Entire cluster only                                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Comparison:                                                         │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Feature         │ pg_dump          │ pg_basebackup          │  │
│  │  ────────────────────────────────────────────────────────────  │  │
│  │  Speed           │ Slow             │ Fast                   │  │
│  │  Cross-version   │ Yes              │ No                     │  │
│  │  Selective       │ Yes              │ No (full cluster)      │  │
│  │  PITR            │ No               │ Yes                    │  │
│  │  Size            │ Compressed SQL   │ Full data size         │  │
│  │  Online          │ Yes              │ Yes                    │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. pg_dump and pg_dumpall

### Basic pg_dump Usage

```bash
# Dump single database
pg_dump -h localhost -U postgres dbname > backup.sql

# Custom format (compressed, parallel restore)
pg_dump -Fc dbname > backup.dump

# Directory format (parallel dump and restore)
pg_dump -Fd dbname -j 4 -f backup_dir/

# Tar format
pg_dump -Ft dbname > backup.tar

# Include CREATE DATABASE statement
pg_dump -C dbname > backup_with_create.sql

# Schema only (no data)
pg_dump -s dbname > schema.sql

# Data only (no schema)
pg_dump -a dbname > data.sql

# Specific tables
pg_dump -t users -t orders dbname > tables.sql

# Exclude tables
pg_dump -T logs -T temp_* dbname > backup.sql

# Specific schema
pg_dump -n public dbname > public_schema.sql
```

### pg_dumpall

```bash
# Dump all databases (including roles and tablespaces)
pg_dumpall > full_cluster.sql

# Roles only
pg_dumpall --roles-only > roles.sql

# Tablespaces only
pg_dumpall --tablespaces-only > tablespaces.sql

# Globals (roles + tablespaces) + per-database dumps
pg_dumpall --globals-only > globals.sql
for db in $(psql -At -c "SELECT datname FROM pg_database WHERE datistemplate = false"); do
    pg_dump -Fc "$db" > "${db}.dump"
done
```

### Restore from pg_dump

```bash
# Plain SQL format
psql -d newdb < backup.sql

# Create database and restore
psql -c "CREATE DATABASE newdb;" postgres
psql -d newdb < backup.sql

# Custom format
pg_restore -d newdb backup.dump

# Parallel restore
pg_restore -d newdb -j 4 backup.dump

# Directory format (parallel)
pg_restore -d newdb -j 8 backup_dir/

# Create database during restore
pg_restore -C -d postgres backup.dump

# List contents of dump
pg_restore -l backup.dump

# Selective restore using list
pg_restore -l backup.dump > toc.txt
# Edit toc.txt to comment out unwanted items
pg_restore -d newdb -L toc.txt backup.dump

# Schema only
pg_restore -s -d newdb backup.dump

# Data only
pg_restore -a -d newdb backup.dump

# Single table
pg_restore -d newdb -t users backup.dump
```

---

## 3. pg_basebackup

### Physical Backup

```bash
# Basic backup
pg_basebackup -D /backup/base -Fp -Xs -P

# Options:
# -D directory     Backup destination
# -Fp              Plain format
# -Ft              Tar format
# -Xs              Stream WAL during backup
# -P               Show progress
# -c fast          Fast checkpoint (immediate)
# -R               Create recovery configuration

# Remote backup
pg_basebackup \
    -h primary.example.com \
    -U replicator \
    -D /backup/base \
    -Fp \
    -Xs \
    -P \
    -R

# Compressed tar backup
pg_basebackup \
    -D /backup \
    -Ft \
    -z \      # gzip compression
    -Xs \
    -P

# With replication slot
pg_basebackup \
    -D /backup/base \
    -S backup_slot \
    -Xs \
    -P
```

### Backup to Standby

```bash
# Create standby server directly
pg_basebackup \
    -h primary.example.com \
    -D /var/lib/postgresql/15/main \
    -U replicator \
    -Fp \
    -Xs \
    -P \
    -R  # Creates standby.signal and postgresql.auto.conf

# Start standby
pg_ctl start -D /var/lib/postgresql/15/main
```

---

## 4. Point-in-Time Recovery (PITR)

### WAL Archiving Setup

```ini
# postgresql.conf on primary

# Enable archiving
archive_mode = on
archive_command = 'cp %p /archive/%f'

# Or with pgBackRest
archive_command = 'pgbackrest --stanza=main archive-push %p'

# Or with Barman
archive_command = 'barman-wal-archive backup-server main %p'

# WAL level
wal_level = replica

# Retention (alternative to slots)
wal_keep_size = 1GB
```

### PITR Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Point-in-Time Recovery                            │
│                                                                      │
│  Disaster at                                                         │
│  2024-01-15 14:30                                                   │
│       ▼                                                              │
│  ──────────────────────────────────────────────────────────────     │
│  Base Backup     WAL Segments              Recovery Target          │
│  (Jan 10)        (Jan 10-15)               (Jan 15 14:00)           │
│       │                                         │                    │
│       ▼                                         ▼                    │
│  [████████] + [WAL][WAL][WAL]...[WAL] ──► [████████████████]        │
│                                                                      │
│  Steps:                                                              │
│  1. Restore base backup                                              │
│  2. Configure recovery target (time, XID, or LSN)                   │
│  3. Ensure WAL archive is accessible                                │
│  4. Start PostgreSQL                                                 │
│  5. Recovery replays WAL until target                               │
│  6. Database opens in recovered state                               │
└─────────────────────────────────────────────────────────────────────┘
```

### Recovery Configuration

```ini
# postgresql.conf (PostgreSQL 12+)

# How to get archived WAL
restore_command = 'cp /archive/%f %p'

# Recovery target (choose one)
recovery_target_time = '2024-01-15 14:00:00'
# recovery_target_xid = '12345678'
# recovery_target_lsn = '0/1000000'
# recovery_target_name = 'before_migration'  # Named restore point

# Whether to include the target transaction
recovery_target_inclusive = true

# What to do when target is reached
recovery_target_action = 'promote'  # or 'pause', 'shutdown'
```

### Performing PITR

```bash
# 1. Stop PostgreSQL (if running)
pg_ctl stop -D /var/lib/postgresql/15/main

# 2. Move or remove current data
mv /var/lib/postgresql/15/main /var/lib/postgresql/15/main.old

# 3. Restore base backup
cp -r /backup/base /var/lib/postgresql/15/main

# 4. Create recovery signal
touch /var/lib/postgresql/15/main/recovery.signal

# 5. Configure recovery in postgresql.conf
cat >> /var/lib/postgresql/15/main/postgresql.conf << EOF
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2024-01-15 14:00:00'
recovery_target_action = 'promote'
EOF

# 6. Start PostgreSQL
pg_ctl start -D /var/lib/postgresql/15/main

# 7. Check recovery status
psql -c "SELECT pg_is_in_recovery();"  # true during recovery

# 8. Verify data
psql -c "SELECT MAX(created_at) FROM important_table;"
```

---

## 5. pgBackRest

### Configuration

```ini
# /etc/pgbackrest/pgbackrest.conf

[global]
repo1-path=/backup/pgbackrest
repo1-retention-full=2
repo1-retention-diff=4
log-level-console=info
log-level-file=debug
start-fast=y
process-max=4

[main]
pg1-path=/var/lib/postgresql/15/main
pg1-port=5432
```

### pgBackRest Commands

```bash
# Create stanza (initialization)
pgbackrest --stanza=main stanza-create

# Check configuration
pgbackrest --stanza=main check

# Full backup
pgbackrest --stanza=main --type=full backup

# Differential backup
pgbackrest --stanza=main --type=diff backup

# Incremental backup
pgbackrest --stanza=main --type=incr backup

# List backups
pgbackrest --stanza=main info

# Restore
pgbackrest --stanza=main restore

# Restore to point in time
pgbackrest --stanza=main \
    --target='2024-01-15 14:00:00' \
    --target-action=promote \
    restore

# Restore specific database
pgbackrest --stanza=main --db-include=mydb restore
```

### Archive Push/Get

```ini
# postgresql.conf
archive_command = 'pgbackrest --stanza=main archive-push %p'
restore_command = 'pgbackrest --stanza=main archive-get %f "%p"'
```

---

## 6. Barman

### Barman Configuration

```ini
# /etc/barman.conf

[barman]
barman_home = /var/lib/barman
configuration_files_directory = /etc/barman.d
barman_user = barman
log_file = /var/log/barman/barman.log
log_level = INFO

# /etc/barman.d/main.conf
[main]
description = "Main PostgreSQL Server"
ssh_command = ssh postgres@primary.example.com
conninfo = host=primary.example.com user=barman dbname=postgres
backup_method = rsync
archiver = on
backup_directory = /var/lib/barman/main
retention_policy = RECOVERY WINDOW OF 7 DAYS
```

### Barman Commands

```bash
# Check configuration
barman check main

# List servers
barman list-server

# Create backup
barman backup main

# List backups
barman list-backup main

# Show backup details
barman show-backup main latest

# Restore
barman recover main latest /var/lib/postgresql/15/main

# PITR restore
barman recover main latest /var/lib/postgresql/15/main \
    --target-time "2024-01-15 14:00:00"

# Delete old backups
barman delete main oldest
```

---

## 7. Backup Best Practices

### Backup Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Recommended Backup Strategy                       │
│                                                                      │
│  Frequency:                                                          │
│  • Full backup: Weekly (Sunday night)                               │
│  • Differential: Daily                                               │
│  • WAL archiving: Continuous                                         │
│                                                                      │
│  Retention:                                                          │
│  • Full backups: 4 weeks                                            │
│  • WAL archives: Until oldest full backup +1 week                   │
│                                                                      │
│  Testing:                                                            │
│  • Monthly restore test to separate server                          │
│  • Verify data integrity                                            │
│                                                                      │
│  Storage:                                                            │
│  • Local + remote (cloud storage)                                   │
│  • Encrypted at rest and in transit                                 │
│  • Monitor backup success/failure                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Monitoring Backups

```sql
-- Check WAL archiving status
SELECT * FROM pg_stat_archiver;

-- Last archived WAL
SELECT last_archived_wal, last_archived_time
FROM pg_stat_archiver;

-- Archive lag
SELECT now() - last_archived_time AS archive_lag
FROM pg_stat_archiver;

-- Ready to archive files
SELECT COUNT(*) FROM pg_ls_dir('pg_wal/archive_status')
WHERE pg_ls_dir LIKE '%.ready';
```

### Backup Verification

```bash
# Verify pg_dump backup
pg_restore --list backup.dump > /dev/null && echo "Valid"

# Test restore to temp database
createdb test_restore
pg_restore -d test_restore backup.dump
psql -d test_restore -c "SELECT COUNT(*) FROM users;"
dropdb test_restore

# Verify pgBackRest backup
pgbackrest --stanza=main verify

# Check backup integrity
pgbackrest --stanza=main --set=latest info --output=json | jq '.backup[-1]'
```

---

## 8. Disaster Recovery Runbook

### Recovery Steps

```
1. ASSESS SITUATION
   - What failed? (disk, server, database corruption)
   - What's the recovery point objective (RPO)?
   - What's the recovery time objective (RTO)?

2. PREPARE RECOVERY ENVIRONMENT
   - New server or repaired server
   - Same PostgreSQL version
   - Sufficient disk space

3. RESTORE BASE BACKUP
   pgbackrest --stanza=main restore
   # or
   pg_basebackup -D /data -h backup_server

4. CONFIGURE RECOVERY TARGET
   # Edit postgresql.conf
   restore_command = '...'
   recovery_target_time = '...'

5. START RECOVERY
   pg_ctl start -D /data

6. VERIFY RECOVERY
   psql -c "SELECT pg_is_in_recovery();"
   # Check critical data

7. PROMOTE IF NEEDED
   pg_ctl promote -D /data

8. UPDATE CONNECTIONS
   - Update DNS/load balancer
   - Notify applications

9. POST-RECOVERY
   - Take fresh backup
   - Review and document incident
```

---

## Summary

| Tool | Use Case |
|------|----------|
| pg_dump | Logical, portable, selective |
| pg_basebackup | Physical, fast, PITR capable |
| pgBackRest | Enterprise backup solution |
| Barman | Another enterprise solution |
| WAL archiving | Continuous, enables PITR |

---

## Further Reading

- PostgreSQL Backup and Restore documentation
- pgBackRest User Guide
- Barman documentation
- Continuous Archiving and PITR
