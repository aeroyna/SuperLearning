# Write-Ahead Logging (WAL) and Checkpoints

## Learning Objectives
- Understand WAL fundamentals and importance
- Master checkpoint behavior and tuning
- Configure WAL for performance and durability
- Monitor and troubleshoot WAL-related issues

---

## 1. WAL Fundamentals

### Write-Ahead Logging Concept

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Write-Ahead Logging (WAL)                         │
│                                                                      │
│  "Changes must be written to the log BEFORE being applied to data"  │
│                                                                      │
│  Transaction: UPDATE accounts SET balance = balance - 100           │
│                                                                      │
│  Step 1: Write to WAL buffer                                         │
│  ┌────────────────────────────────────────────────────────┐         │
│  │ LSN: 0/3000000 | XID: 1234 | Operation: UPDATE         │         │
│  │ Table: accounts | TID: (0,5) | Old: 500 | New: 400     │         │
│  └────────────────────────────────────────────────────────┘         │
│                         │                                            │
│                         ▼                                            │
│  Step 2: Flush WAL to disk (at commit)                              │
│  ┌────────────────────────────────────────────────────────┐         │
│  │              WAL Segment File (16MB)                   │         │
│  │  000000010000000000000001                               │         │
│  └────────────────────────────────────────────────────────┘         │
│                         │                                            │
│                         ▼                                            │
│  Step 3: Apply change to data page (eventually)                     │
│  ┌────────────────────────────────────────────────────────┐         │
│  │              Shared Buffer (Data Page)                 │         │
│  └────────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────────┘
```

### Why WAL?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Benefits of WAL                                   │
│                                                                      │
│  1. CRASH RECOVERY                                                   │
│     ┌─────────────────────────────────────────────────────────┐     │
│     │  Crash occurs → Replay WAL from last checkpoint         │     │
│     │  Committed transactions are recovered                   │     │
│     │  Uncommitted transactions are rolled back               │     │
│     └─────────────────────────────────────────────────────────┘     │
│                                                                      │
│  2. PERFORMANCE                                                      │
│     ┌─────────────────────────────────────────────────────────┐     │
│     │  Sequential WAL writes vs Random data page writes       │     │
│     │  Batch data page writes (background writer)             │     │
│     │  Reduces disk I/O significantly                         │     │
│     └─────────────────────────────────────────────────────────┘     │
│                                                                      │
│  3. REPLICATION                                                      │
│     ┌─────────────────────────────────────────────────────────┐     │
│     │  Stream WAL to replicas                                 │     │
│     │  Point-in-time recovery (PITR)                          │     │
│     │  Continuous archiving                                   │     │
│     └─────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────┘
```

### WAL Record Structure

```sql
-- View WAL location
SELECT pg_current_wal_lsn();
-- Result: 0/3000000

-- WAL location format: segment/offset
-- Segment: Which 16MB file
-- Offset: Position within file

-- View WAL statistics
SELECT * FROM pg_stat_wal;

-- Columns include:
-- wal_records: Total records generated
-- wal_fpi: Full page images written
-- wal_bytes: Total bytes generated
-- wal_buffers_full: Times WAL buffers were full
-- wal_write: Number of WAL writes
-- wal_sync: Number of WAL syncs
```

---

## 2. WAL Configuration

### Essential Settings

```ini
# postgresql.conf

# WAL Level
# minimal: Crash recovery only
# replica: Streaming replication (default)
# logical: Logical replication
wal_level = replica

# WAL Segment Size (compile-time, default 16MB)
# Larger segments = less file switching
# Cannot change without reinitializing cluster

# Number of WAL segments to keep
min_wal_size = 1GB      # Minimum WAL disk usage
max_wal_size = 4GB      # Maximum before aggressive checkpointing

# WAL buffers (shared memory)
wal_buffers = 64MB      # -1 = auto (3% of shared_buffers, max 16MB)

# Synchronous commit
synchronous_commit = on  # on, off, local, remote_write, remote_apply
```

### Durability vs Performance

```ini
# MAXIMUM DURABILITY (default)
synchronous_commit = on
fsync = on
full_page_writes = on
wal_sync_method = fdatasync

# PERFORMANCE (less durable)
synchronous_commit = off     # Don't wait for WAL flush
# Risk: May lose recent transactions (up to wal_writer_delay)

# DANGEROUS (never in production)
# fsync = off                # Corruption risk on crash!
```

### WAL Sync Methods

```sql
-- View current sync method
SHOW wal_sync_method;

-- Options (platform dependent):
-- open_datasync: O_DSYNC on open()
-- fdatasync: fdatasync() after each write (Linux default)
-- fsync: fsync() after each write
-- fsync_writethrough: fsync with write-through
-- open_sync: O_SYNC on open()

-- Test sync methods with pg_test_fsync
```

---

## 3. Checkpoints

### Checkpoint Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Checkpoint Operation                              │
│                                                                      │
│  Before Checkpoint:                                                  │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Shared Buffers                                              │    │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                    │    │
│  │  │Dirty│ │Clean│ │Dirty│ │Dirty│ │Clean│  ...               │    │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Checkpoint Steps:                                                   │
│  1. Mark checkpoint start in WAL                                     │
│  2. Flush all dirty buffers to disk                                  │
│  3. Sync all data files                                              │
│  4. Write checkpoint record to WAL                                   │
│  5. Update pg_control with checkpoint location                       │
│                                                                      │
│  After Checkpoint:                                                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Shared Buffers                                              │    │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                    │    │
│  │  │Clean│ │Clean│ │Clean│ │Clean│ │Clean│  ...               │    │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  → WAL before checkpoint can be recycled/archived                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Checkpoint Triggers

```ini
# postgresql.conf

# Time-based checkpoint
checkpoint_timeout = 5min    # Maximum time between checkpoints

# WAL-based checkpoint
max_wal_size = 4GB           # Checkpoint when WAL reaches this size

# Spread checkpoint I/O
checkpoint_completion_target = 0.9  # Spread over 90% of checkpoint interval

# Warning threshold
checkpoint_warning = 30s     # Warn if checkpoints more frequent
```

### Checkpoint Configuration

```sql
-- View checkpoint statistics
SELECT * FROM pg_stat_bgwriter;

-- Key columns:
-- checkpoints_timed: Scheduled checkpoints
-- checkpoints_req: Requested checkpoints (WAL size trigger)
-- checkpoint_write_time: Time spent writing
-- checkpoint_sync_time: Time spent syncing
-- buffers_checkpoint: Buffers written during checkpoints

-- Ideally: checkpoints_timed >> checkpoints_req
-- Many checkpoints_req = max_wal_size too low

-- Manual checkpoint (admin only)
CHECKPOINT;
```

### Spread Checkpoints

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Spread Checkpoint I/O                             │
│                                                                      │
│  checkpoint_completion_target = 0.9                                  │
│  checkpoint_timeout = 5min                                           │
│                                                                      │
│  Time: 0    1    2    3    4    4.5   5 min                         │
│        │    │    │    │    │    │     │                              │
│        ▼    ▼    ▼    ▼    ▼    ▼     ▼                              │
│        ┌────────────────────────┐     ┌──                            │
│        │   Spread writes over   │     │Next                          │
│        │   4.5 min (90% of 5)   │     │CP                            │
│        └────────────────────────┘     └──                            │
│        │◄──────────────────────►│                                    │
│         Checkpoint write window                                      │
│                                                                      │
│  Benefits:                                                           │
│  - Reduced I/O spikes                                                │
│  - Better query performance during checkpoint                        │
│  - More predictable latency                                          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Full Page Writes

### Why Full Page Writes?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Partial Write Problem                             │
│                                                                      │
│  PostgreSQL page: 8KB                                                │
│  Filesystem block: typically 4KB                                     │
│                                                                      │
│  Scenario: Writing 8KB page, crash after first 4KB                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  8KB Page                                                    │    │
│  │  ┌─────────────────────┬─────────────────────┐              │    │
│  │  │   First 4KB         │   Second 4KB        │              │    │
│  │  │   (NEW data)        │   (OLD data)        │  ← TORN!     │    │
│  │  └─────────────────────┴─────────────────────┘              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Solution: Full Page Write (FPW)                                     │
│  - First modification after checkpoint writes entire page to WAL    │
│  - Recovery replaces torn page with WAL copy                         │
│  - Increases WAL size but ensures consistency                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Configuration

```ini
# postgresql.conf

# Enable full page writes (default: on)
full_page_writes = on

# Compress full page images
wal_compression = on         # Reduces WAL size by ~50%
                              # Small CPU overhead

# Disable only with battery-backed write cache
# Or ZFS/similar with atomic writes
```

### Monitoring FPW

```sql
-- Full page images in WAL
SELECT wal_fpi FROM pg_stat_wal;

-- High FPW rate indicates:
-- - Frequent checkpoints
-- - Large working set
-- - Need to tune checkpoint interval

-- View WAL generation rate
SELECT
    pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0') / (1024*1024*1024) AS wal_generated_gb,
    pg_wal_lsn_diff(pg_current_wal_lsn(), '0/0') /
        EXTRACT(EPOCH FROM (now() - pg_postmaster_start_time())) / 1024 AS wal_rate_kb_per_sec;
```

---

## 5. WAL Archiving

### Archive Configuration

```ini
# postgresql.conf

# Enable archiving
archive_mode = on

# Archive command (runs for each completed WAL segment)
archive_command = 'cp %p /archive/%f'
# %p = full path to WAL file
# %f = WAL filename only

# Or with compression
archive_command = 'gzip < %p > /archive/%f.gz'

# Archive timeout (force switch even if segment not full)
archive_timeout = 300        # 5 minutes
```

### Archive Examples

```bash
# Copy to remote server
archive_command = 'rsync -a %p backup@archive-server:/archive/%f'

# Copy to S3
archive_command = 'aws s3 cp %p s3://bucket/wal/%f'

# Using pgBackRest
archive_command = 'pgbackrest --stanza=main archive-push %p'

# Using Barman
archive_command = 'barman-wal-archive backup-server main %p'
```

### Monitoring Archiving

```sql
-- Archive status
SELECT * FROM pg_stat_archiver;

-- Columns:
-- archived_count: Successfully archived
-- last_archived_wal: Last archived segment
-- last_archived_time: When
-- failed_count: Failed attempts
-- last_failed_wal: Last failed segment

-- Check archive lag
SELECT
    last_archived_wal,
    last_archived_time,
    now() - last_archived_time AS archive_lag
FROM pg_stat_archiver;

-- Ready-to-archive files
SELECT COUNT(*) FROM pg_ls_dir('pg_wal/archive_status')
WHERE pg_ls_dir LIKE '%.ready';
```

---

## 6. Point-in-Time Recovery (PITR)

### PITR Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Point-in-Time Recovery                            │
│                                                                      │
│  Base Backup              WAL Archive              Recovery Target  │
│  (Sunday 00:00)           (Continuous)             (Wed 14:30)      │
│       │                        │                        │            │
│       ▼                        ▼                        ▼            │
│  [████████] + [WAL][WAL][WAL][WAL][WAL][WAL] = [████████████████]   │
│                                                                      │
│  Steps:                                                              │
│  1. Restore base backup                                              │
│  2. Configure recovery settings                                      │
│  3. Start PostgreSQL                                                 │
│  4. WAL replay until target                                          │
│  5. Database opens for connections                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Recovery Configuration

```ini
# postgresql.conf (PostgreSQL 12+)

# Restore command (how to retrieve archived WAL)
restore_command = 'cp /archive/%f %p'

# Recovery target options (choose one)
recovery_target_time = '2024-01-15 14:30:00'
# recovery_target_xid = '1234'
# recovery_target_lsn = '0/1000000'
# recovery_target_name = 'my_restore_point'

# What to do after reaching target
recovery_target_action = 'promote'  # pause, promote, shutdown

# Include/exclude target transaction
recovery_target_inclusive = true
```

### Create Restore Point

```sql
-- Create named restore point
SELECT pg_create_restore_point('before_migration');

-- Use in recovery
-- recovery_target_name = 'before_migration'
```

---

## 7. WAL Monitoring

### Key Metrics

```sql
-- Current WAL location
SELECT pg_current_wal_lsn();

-- WAL generation rate
SELECT
    pg_size_pretty(pg_wal_lsn_diff(
        pg_current_wal_lsn(),
        '0/0'
    )) AS total_wal_generated;

-- WAL write vs sync time
SELECT
    wal_write_time,
    wal_sync_time,
    wal_write_time + wal_sync_time AS total_time
FROM pg_stat_wal;

-- Checkpoint frequency
SELECT
    checkpoints_timed,
    checkpoints_req,
    ROUND(100.0 * checkpoints_req /
        NULLIF(checkpoints_timed + checkpoints_req, 0), 2) AS pct_forced
FROM pg_stat_bgwriter;
```

### WAL Files on Disk

```sql
-- List WAL files
SELECT * FROM pg_ls_waldir() ORDER BY modification DESC LIMIT 10;

-- WAL directory size
SELECT pg_size_pretty(sum(size)) FROM pg_ls_waldir();

-- Count WAL files
SELECT COUNT(*) FROM pg_ls_waldir();
```

### Performance Analysis

```sql
-- WAL efficiency
SELECT
    wal_records,
    wal_fpi,
    ROUND(100.0 * wal_fpi / NULLIF(wal_records, 0), 2) AS fpi_pct,
    pg_size_pretty(wal_bytes) AS wal_size,
    pg_size_pretty(wal_bytes / NULLIF(wal_records, 0)) AS avg_record_size
FROM pg_stat_wal;

-- High fpi_pct suggests:
-- - Too frequent checkpoints
-- - May benefit from wal_compression
```

---

## 8. Configuration Templates

### High Durability (Financial)

```ini
# Maximum data protection
wal_level = replica
synchronous_commit = on
fsync = on
full_page_writes = on
wal_compression = on

checkpoint_timeout = 15min
max_wal_size = 8GB
checkpoint_completion_target = 0.9

archive_mode = on
archive_command = 'pgbackrest --stanza=prod archive-push %p'
```

### High Performance (Analytics)

```ini
# Optimized for throughput
wal_level = minimal
synchronous_commit = off
full_page_writes = on
wal_compression = on

checkpoint_timeout = 30min
max_wal_size = 16GB
checkpoint_completion_target = 0.9

# No archiving needed
archive_mode = off
```

### Replication Ready

```ini
# For streaming replication
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1GB

# For logical replication
# wal_level = logical

synchronous_commit = on
archive_mode = on
archive_command = 'cp %p /archive/%f'
```

---

## 9. Troubleshooting

### Common Issues

```sql
-- Issue: WAL accumulation (disk full)
-- Check why WAL not being removed
SELECT * FROM pg_replication_slots;
-- Inactive slots prevent WAL removal
SELECT pg_drop_replication_slot('inactive_slot');

-- Check archive status
SELECT * FROM pg_stat_archiver;
-- Failed archiving prevents WAL removal

-- Manual WAL cleanup (dangerous!)
-- Only if you understand the implications
SELECT pg_switch_wal();  -- Force segment switch
```

### Checkpoint Performance

```sql
-- Checkpoint taking too long?
-- Check buffers being written
SELECT
    buffers_checkpoint,
    buffers_clean,
    buffers_backend,
    buffers_backend_fsync
FROM pg_stat_bgwriter;

-- High buffers_backend = background writer too slow
-- Increase bgwriter_lru_maxpages
-- Increase checkpoint_timeout for fewer checkpoints
```

### WAL Write Latency

```sql
-- Sync time too high?
SELECT
    wal_write,
    wal_sync,
    wal_write_time,
    wal_sync_time,
    ROUND(wal_sync_time / NULLIF(wal_sync, 0), 2) AS avg_sync_time_ms
FROM pg_stat_wal;

-- High sync time indicates storage bottleneck
-- Consider: faster storage, battery-backed cache
```

---

## Summary

| Setting | Default | Purpose |
|---------|---------|---------|
| wal_level | replica | WAL detail level |
| synchronous_commit | on | Durability guarantee |
| checkpoint_timeout | 5min | Max checkpoint interval |
| max_wal_size | 1GB | WAL size checkpoint trigger |
| checkpoint_completion_target | 0.9 | Spread checkpoint I/O |
| full_page_writes | on | Prevent torn pages |
| wal_compression | off | Compress FPW |

---

## Further Reading

- PostgreSQL WAL Internals documentation
- "PostgreSQL 14 Internals" - WAL chapters
- pg_waldump utility for WAL inspection
