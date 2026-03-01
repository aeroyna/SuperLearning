# Backup Strategies

## Full Backups

```
┌─────────────────────────────────────────────────────────────────┐
│              Full Backup Strategy                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONCEPT                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Database ────────────► Complete Copy                      │ │
│  │  [A B C D E]             [A B C D E]                       │ │
│  │                                                             │ │
│  │  • Captures everything                                     │ │
│  │  • Self-contained for restore                              │ │
│  │  • Largest backup size                                     │ │
│  │  • Longest backup time                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL (pg_dump)                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Logical backup (SQL format)                              │ │
│  │ pg_dump -h localhost -U postgres mydb > backup.sql         │ │
│  │                                                             │ │
│  │ # Custom format (compressed, parallel restore)             │ │
│  │ pg_dump -Fc -h localhost -U postgres mydb > backup.dump    │ │
│  │                                                             │ │
│  │ # Directory format (parallel backup)                       │ │
│  │ pg_dump -Fd -j 4 -h localhost -U postgres mydb -f backup/  │ │
│  │                                                             │ │
│  │ # Full cluster backup                                      │ │
│  │ pg_dumpall -h localhost -U postgres > full_backup.sql      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MYSQL (mysqldump)                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Single database                                          │ │
│  │ mysqldump -u root -p mydb > backup.sql                     │ │
│  │                                                             │ │
│  │ # All databases                                            │ │
│  │ mysqldump -u root -p --all-databases > full_backup.sql     │ │
│  │                                                             │ │
│  │ # With routines and triggers                               │ │
│  │ mysqldump -u root -p --routines --triggers mydb > backup.sql │
│  │                                                             │ │
│  │ # Consistent snapshot (InnoDB)                             │ │
│  │ mysqldump -u root -p --single-transaction mydb > backup.sql │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PHYSICAL BACKUPS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PostgreSQL (pg_basebackup):                                │ │
│  │ pg_basebackup -h localhost -U replicator -D /backup/base \ │ │
│  │   -Fp -Xs -P                                               │ │
│  │                                                             │ │
│  │ MySQL (xtrabackup):                                        │ │
│  │ xtrabackup --backup --target-dir=/backup/full \            │ │
│  │   --user=root --password=secret                            │ │
│  │                                                             │ │
│  │ Physical backups are faster for large databases            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Incremental Backups

```
┌─────────────────────────────────────────────────────────────────┐
│              Incremental Backup Strategy                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONCEPT                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Day 1: Full    [A B C D E]                                │ │
│  │  Day 2: Incr    [  F G    ]  (changes since Day 1)        │ │
│  │  Day 3: Incr    [    H    ]  (changes since Day 2)        │ │
│  │  Day 4: Incr    [      I J]  (changes since Day 3)        │ │
│  │                                                             │ │
│  │  Restore requires: Full + all incrementals in order       │ │
│  │                                                             │ │
│  │  ✓ Small backup size                                       │ │
│  │  ✓ Fast backup time                                        │ │
│  │  ✗ Complex restore                                         │ │
│  │  ✗ Chain dependency                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL (WAL Archiving)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # postgresql.conf                                          │ │
│  │ wal_level = replica                                        │ │
│  │ archive_mode = on                                          │ │
│  │ archive_command = 'cp %p /archive/%f'                      │ │
│  │                                                             │ │
│  │ # Base backup + WAL files = Point-in-Time Recovery        │ │
│  │ # WAL files are the "incrementals"                        │ │
│  │                                                             │ │
│  │ # With pgBackRest (recommended):                          │ │
│  │ pgbackrest --stanza=main --type=incr backup               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MYSQL (xtrabackup)                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Full backup first                                        │ │
│  │ xtrabackup --backup --target-dir=/backup/full              │ │
│  │                                                             │ │
│  │ # Incremental backup                                       │ │
│  │ xtrabackup --backup --target-dir=/backup/inc1 \            │ │
│  │   --incremental-basedir=/backup/full                       │ │
│  │                                                             │ │
│  │ # Next incremental                                         │ │
│  │ xtrabackup --backup --target-dir=/backup/inc2 \            │ │
│  │   --incremental-basedir=/backup/inc1                       │ │
│  │                                                             │ │
│  │ # Prepare for restore (apply incrementals)                │ │
│  │ xtrabackup --prepare --apply-log-only --target-dir=/backup/full │
│  │ xtrabackup --prepare --apply-log-only --target-dir=/backup/full \ │
│  │   --incremental-dir=/backup/inc1                           │ │
│  │ xtrabackup --prepare --target-dir=/backup/full \           │ │
│  │   --incremental-dir=/backup/inc2                           │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Differential Backups

```
┌─────────────────────────────────────────────────────────────────┐
│              Differential Backup Strategy                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONCEPT                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Day 1: Full    [A B C D E]                                │ │
│  │  Day 2: Diff    [  F G    ]  (changes since Full)         │ │
│  │  Day 3: Diff    [  F G H  ]  (changes since Full)         │ │
│  │  Day 4: Diff    [  F G H I J] (changes since Full)        │ │
│  │                                                             │ │
│  │  Restore requires: Full + latest differential only        │ │
│  │                                                             │ │
│  │  ✓ Simpler restore than incremental                        │ │
│  │  ✓ Only need Full + one Diff                               │ │
│  │  ✗ Diff grows over time                                    │ │
│  │  ✗ Larger than incremental                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMPARISON                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Method      │ Backup Size │ Backup Time │ Restore Time   │ │
│  │  ────────────┼─────────────┼─────────────┼────────────────│ │
│  │  Full        │ Largest     │ Longest     │ Shortest       │ │
│  │  Incremental │ Smallest    │ Shortest    │ Longest        │ │
│  │  Differential│ Medium      │ Medium      │ Medium         │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TYPICAL SCHEDULE                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Weekly Full + Daily Differential:                          │ │
│  │                                                             │ │
│  │ Sun    Mon    Tue    Wed    Thu    Fri    Sat              │ │
│  │ Full   Diff   Diff   Diff   Diff   Diff   Diff             │ │
│  │                                                             │ │
│  │ Restore Wednesday: Sun Full + Wed Diff                     │ │
│  │                                                             │ │
│  │ Weekly Full + Daily Incremental:                           │ │
│  │                                                             │ │
│  │ Sun    Mon    Tue    Wed    Thu    Fri    Sat              │ │
│  │ Full   Incr   Incr   Incr   Incr   Incr   Incr             │ │
│  │                                                             │ │
│  │ Restore Wednesday: Sun Full + Mon + Tue + Wed Incr        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Logical vs Physical Backups

```
┌─────────────────────────────────────────────────────────────────┐
│              Logical vs Physical Backups                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOGICAL BACKUPS                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ What: SQL statements or data export                        │ │
│  │ Tools: pg_dump, mysqldump, mongodump                       │ │
│  │                                                             │ │
│  │ Advantages:                                                 │ │
│  │ • Portable across versions                                 │ │
│  │ • Human-readable (SQL format)                              │ │
│  │ • Can restore individual tables                           │ │
│  │ • Works across different OS                                │ │
│  │                                                             │ │
│  │ Disadvantages:                                              │ │
│  │ • Slower for large databases                               │ │
│  │ • Higher CPU usage                                         │ │
│  │ • Larger backup size (uncompressed)                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PHYSICAL BACKUPS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ What: Copy of data files                                   │ │
│  │ Tools: pg_basebackup, xtrabackup, filesystem snapshots    │ │
│  │                                                             │ │
│  │ Advantages:                                                 │ │
│  │ • Faster backup and restore                                │ │
│  │ • Lower CPU overhead                                       │ │
│  │ • Supports point-in-time recovery                         │ │
│  │ • Exact copy of database                                   │ │
│  │                                                             │ │
│  │ Disadvantages:                                              │ │
│  │ • Version-specific                                         │ │
│  │ • Platform-dependent                                       │ │
│  │ • All-or-nothing restore                                   │ │
│  │ • Requires same directory structure                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  WHEN TO USE EACH                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Use Logical When:                                          │ │
│  │ • Migrating between versions                               │ │
│  │ • Need to restore specific tables                         │ │
│  │ • Moving between platforms                                 │ │
│  │ • Database is small (<100GB)                              │ │
│  │                                                             │ │
│  │ Use Physical When:                                         │ │
│  │ • Large databases (>100GB)                                │ │
│  │ • Need point-in-time recovery                             │ │
│  │ • Fast restore is critical                                 │ │
│  │ • Setting up replicas                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Backup Scheduling

```
┌─────────────────────────────────────────────────────────────────┐
│              Backup Scheduling Strategies                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GRANDFATHER-FATHER-SON (GFS)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Grandfather (Monthly): Keep 12 months                      │ │
│  │ ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐ │
│  │ │Jan │Feb │Mar │Apr │May │Jun │Jul │Aug │Sep │Oct │Nov │Dec │ │
│  │ └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘ │
│  │                                                             │ │
│  │ Father (Weekly): Keep 4 weeks                              │ │
│  │ ┌────────┬────────┬────────┬────────┐                      │ │
│  │ │ Week 1 │ Week 2 │ Week 3 │ Week 4 │                      │ │
│  │ └────────┴────────┴────────┴────────┘                      │ │
│  │                                                             │ │
│  │ Son (Daily): Keep 7 days                                   │ │
│  │ ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐                │ │
│  │ │ Mon │ Tue │ Wed │ Thu │ Fri │ Sat │ Sun │                │ │
│  │ └─────┴─────┴─────┴─────┴─────┴─────┴─────┘                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CRON SCHEDULE EXAMPLES                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Daily logical backup at 2 AM                             │ │
│  │ 0 2 * * * /scripts/backup.sh >> /var/log/backup.log 2>&1  │ │
│  │                                                             │ │
│  │ # Weekly full backup on Sunday at 1 AM                    │ │
│  │ 0 1 * * 0 /scripts/full_backup.sh                         │ │
│  │                                                             │ │
│  │ # Hourly incremental                                       │ │
│  │ 0 * * * * /scripts/incremental_backup.sh                  │ │
│  │                                                             │ │
│  │ # Monthly backup on 1st at midnight                       │ │
│  │ 0 0 1 * * /scripts/monthly_backup.sh                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BACKUP SCRIPT EXAMPLE                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ #!/bin/bash                                                │ │
│  │ set -e                                                     │ │
│  │                                                             │ │
│  │ DATE=$(date +%Y%m%d_%H%M%S)                               │ │
│  │ BACKUP_DIR="/backup/postgres"                              │ │
│  │ RETENTION_DAYS=7                                           │ │
│  │                                                             │ │
│  │ # Create backup                                            │ │
│  │ pg_dump -Fc -h localhost -U postgres mydb \               │ │
│  │   > "${BACKUP_DIR}/mydb_${DATE}.dump"                     │ │
│  │                                                             │ │
│  │ # Compress                                                 │ │
│  │ gzip "${BACKUP_DIR}/mydb_${DATE}.dump"                    │ │
│  │                                                             │ │
│  │ # Upload to S3                                             │ │
│  │ aws s3 cp "${BACKUP_DIR}/mydb_${DATE}.dump.gz" \          │ │
│  │   s3://my-backups/postgres/                                │ │
│  │                                                             │ │
│  │ # Cleanup old backups                                      │ │
│  │ find ${BACKUP_DIR} -name "*.dump.gz" \                    │ │
│  │   -mtime +${RETENTION_DAYS} -delete                       │ │
│  │                                                             │ │
│  │ echo "Backup completed: mydb_${DATE}.dump.gz"             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Backup Storage

```
┌─────────────────────────────────────────────────────────────────┐
│              Backup Storage Options                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOCAL STORAGE                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Fast access                                              │ │
│  │ • No network dependency                                    │ │
│  │ • Risk: Same failure domain                               │ │
│  │                                                             │ │
│  │ Use for: Quick restore, temporary staging                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NETWORK STORAGE (NAS/SAN)                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Centralized management                                   │ │
│  │ • Shared across servers                                    │ │
│  │ • RAID protection                                          │ │
│  │                                                             │ │
│  │ Use for: On-premises backup server                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CLOUD OBJECT STORAGE                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ AWS S3 / Google Cloud Storage / Azure Blob                │ │
│  │                                                             │ │
│  │ Storage Classes:                                           │ │
│  │ • Standard: Frequent access                                │ │
│  │ • Infrequent Access: Lower cost, retrieval fee            │ │
│  │ • Glacier/Archive: Lowest cost, slow retrieval            │ │
│  │                                                             │ │
│  │ # Upload to S3 with lifecycle                             │ │
│  │ aws s3 cp backup.dump.gz s3://bucket/backups/ \           │ │
│  │   --storage-class STANDARD_IA                              │ │
│  │                                                             │ │
│  │ # S3 lifecycle policy (JSON)                              │ │
│  │ {                                                          │ │
│  │   "Rules": [{                                             │ │
│  │     "ID": "MoveToGlacier",                                │ │
│  │     "Status": "Enabled",                                  │ │
│  │     "Transitions": [{                                     │ │
│  │       "Days": 30,                                         │ │
│  │       "StorageClass": "GLACIER"                           │ │
│  │     }],                                                    │ │
│  │     "Expiration": { "Days": 365 }                         │ │
│  │   }]                                                       │ │
│  │ }                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BACKUP ENCRYPTION                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Encrypt with GPG                                         │ │
│  │ pg_dump mydb | gzip | gpg -c --passphrase-file key.txt \  │ │
│  │   > backup.sql.gz.gpg                                      │ │
│  │                                                             │ │
│  │ # Decrypt                                                  │ │
│  │ gpg -d --passphrase-file key.txt backup.sql.gz.gpg | \    │ │
│  │   gunzip | psql mydb                                       │ │
│  │                                                             │ │
│  │ # S3 server-side encryption                               │ │
│  │ aws s3 cp backup.dump s3://bucket/ \                      │ │
│  │   --sse aws:kms --sse-kms-key-id alias/backup-key         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Backup Verification

```
┌─────────────────────────────────────────────────────────────────┐
│              Verifying Backup Integrity                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CHECKSUM VERIFICATION                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Generate checksum                                        │ │
│  │ sha256sum backup.dump > backup.dump.sha256                 │ │
│  │                                                             │ │
│  │ # Verify checksum                                          │ │
│  │ sha256sum -c backup.dump.sha256                            │ │
│  │                                                             │ │
│  │ # Verify after upload                                      │ │
│  │ aws s3api head-object --bucket mybucket \                  │ │
│  │   --key backup.dump --checksum-mode ENABLED               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RESTORE TESTING                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # PostgreSQL: Test restore to temporary database          │ │
│  │ createdb test_restore                                      │ │
│  │ pg_restore -d test_restore backup.dump                    │ │
│  │                                                             │ │
│  │ # Verify row counts                                        │ │
│  │ psql -d test_restore -c "SELECT COUNT(*) FROM users;"     │ │
│  │                                                             │ │
│  │ # Cleanup                                                  │ │
│  │ dropdb test_restore                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  AUTOMATED VERIFICATION SCRIPT                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ #!/bin/bash                                                │ │
│  │ set -e                                                     │ │
│  │                                                             │ │
│  │ BACKUP_FILE=$1                                             │ │
│  │ TEST_DB="backup_verify_$(date +%s)"                       │ │
│  │                                                             │ │
│  │ # Create test database                                     │ │
│  │ createdb $TEST_DB                                          │ │
│  │                                                             │ │
│  │ # Restore backup                                           │ │
│  │ pg_restore -d $TEST_DB $BACKUP_FILE                       │ │
│  │                                                             │ │
│  │ # Run verification queries                                 │ │
│  │ USERS=$(psql -t -d $TEST_DB -c "SELECT COUNT(*) FROM users") │
│  │ ORDERS=$(psql -t -d $TEST_DB -c "SELECT COUNT(*) FROM orders") │
│  │                                                             │ │
│  │ # Cleanup                                                  │ │
│  │ dropdb $TEST_DB                                            │ │
│  │                                                             │ │
│  │ # Report                                                   │ │
│  │ echo "Verification complete: $USERS users, $ORDERS orders" │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
