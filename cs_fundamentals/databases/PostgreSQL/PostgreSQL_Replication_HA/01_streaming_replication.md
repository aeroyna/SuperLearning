# Streaming Replication

## Learning Objectives
- Configure primary-standby streaming replication
- Understand synchronous vs asynchronous replication
- Use replication slots effectively
- Monitor replication health and lag

---

## 1. Streaming Replication Overview

### How It Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Streaming Replication Flow                        │
│                                                                      │
│  PRIMARY SERVER                    STANDBY SERVER                    │
│  ┌────────────────────┐           ┌────────────────────┐            │
│  │  Transaction       │           │                    │            │
│  │       ▼            │           │                    │            │
│  │  Write to WAL      │           │                    │            │
│  │       ▼            │           │                    │            │
│  │  WAL Buffer        │           │                    │            │
│  │       ▼            │           │                    │            │
│  │  WAL Segment Files │──Stream──▶│  WAL Receiver     │            │
│  │  (pg_wal/)         │  (TCP)    │       ▼            │            │
│  │                    │           │  WAL Segment Files │            │
│  │  WAL Sender        │           │       ▼            │            │
│  │  Process           │           │  Startup/Recovery  │            │
│  └────────────────────┘           │  Process (replay)  │            │
│                                   │       ▼            │            │
│                                   │  Data Files        │            │
│                                   │  (identical copy)  │            │
│                                   └────────────────────┘            │
│                                                                      │
│  Standby continuously receives and replays WAL from primary         │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Concepts

```
Replication Terms:
• Primary: Read-write server, source of changes
• Standby: Read-only replica, receives WAL stream
• WAL: Write-Ahead Log, transaction changes
• Hot Standby: Standby accepting read queries
• Warm Standby: Standby not accepting queries
• Cascading: Standby replicating to another standby
```

---

## 2. Primary Server Configuration

### postgresql.conf Settings

```ini
# postgresql.conf on PRIMARY

# WAL level (must be replica or logical)
wal_level = replica

# Maximum number of concurrent connections from standbys
max_wal_senders = 10

# Keep enough WAL for standbys (alternative to replication slots)
wal_keep_size = 1GB

# Enable replication slots (recommended)
max_replication_slots = 10

# Archive settings (optional but recommended for PITR)
archive_mode = on
archive_command = 'cp %p /archive/%f'

# Synchronous replication (optional)
# synchronous_standby_names = 'standby1,standby2'
# synchronous_commit = on
```

### pg_hba.conf Authentication

```
# pg_hba.conf on PRIMARY
# Allow replication connections from standby

# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    replication     replicator      192.168.1.0/24         scram-sha-256
host    replication     replicator      standby.example.com    scram-sha-256
```

### Create Replication User

```sql
-- On PRIMARY
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secure_password';

-- Or with connection limit
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'password'
    CONNECTION LIMIT 5;
```

---

## 3. Standby Server Setup

### Using pg_basebackup

```bash
# On STANDBY server

# Stop PostgreSQL if running
sudo systemctl stop postgresql

# Clear data directory
rm -rf /var/lib/postgresql/15/main/*

# Take base backup from primary
pg_basebackup \
    -h primary.example.com \
    -U replicator \
    -D /var/lib/postgresql/15/main \
    -Fp \              # Plain format
    -Xs \              # Stream WAL during backup
    -P \               # Show progress
    -R                 # Create standby.signal and connection info

# -R creates:
# - standby.signal (indicates standby mode)
# - postgresql.auto.conf with primary_conninfo
```

### Manual Standby Configuration

```ini
# postgresql.conf on STANDBY

# Connection to primary
primary_conninfo = 'host=primary.example.com port=5432 user=replicator password=secure_password'

# Slot name (if using replication slot)
primary_slot_name = 'standby1_slot'

# Allow read queries on standby
hot_standby = on

# Feedback to primary (helps with query conflicts)
hot_standby_feedback = on

# Recovery target (optional, for delayed standby)
# recovery_min_apply_delay = '1h'
```

### Standby Signal File

```bash
# Create standby signal (PostgreSQL 12+)
touch /var/lib/postgresql/15/main/standby.signal

# For older versions, create recovery.conf:
# standby_mode = 'on'
# primary_conninfo = '...'
# trigger_file = '/tmp/promote_trigger'
```

---

## 4. Replication Slots

### Why Replication Slots?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Replication Slot Benefits                         │
│                                                                      │
│  Without Slots:                                                      │
│  • WAL segments may be recycled before standby fetches them         │
│  • Standby falls behind → must rebuild from scratch                 │
│  • Need to guess wal_keep_size setting                              │
│                                                                      │
│  With Slots:                                                         │
│  • Primary retains WAL until slot consumer confirms receipt         │
│  • Standby can safely disconnect and reconnect                      │
│  • Guaranteed no WAL loss                                           │
│                                                                      │
│  Risk:                                                               │
│  • Inactive slot causes WAL accumulation (disk full!)               │
│  • Monitor slots, drop unused ones                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating and Managing Slots

```sql
-- Create physical replication slot (on PRIMARY)
SELECT pg_create_physical_replication_slot('standby1_slot');

-- With immediate WAL retention
SELECT pg_create_physical_replication_slot('standby1_slot', true);

-- View slots
SELECT slot_name, slot_type, active, restart_lsn
FROM pg_replication_slots;

-- Check WAL retained by slot
SELECT slot_name,
       pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn) AS retained_bytes
FROM pg_replication_slots;

-- Drop unused slot (IMPORTANT: prevents WAL bloat)
SELECT pg_drop_replication_slot('standby1_slot');
```

### Standby Using Slot

```ini
# postgresql.conf on STANDBY
primary_slot_name = 'standby1_slot'
```

---

## 5. Synchronous Replication

### Configuration

```ini
# postgresql.conf on PRIMARY

# Sync commit level
synchronous_commit = on  # or remote_write, remote_apply

# Standby names (application_name from standby's primary_conninfo)
synchronous_standby_names = 'standby1'

# Multiple standbys (any 1 of 2)
synchronous_standby_names = 'ANY 1 (standby1, standby2)'

# All must confirm
synchronous_standby_names = 'FIRST 2 (standby1, standby2, standby3)'
```

### Synchronous Commit Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    synchronous_commit Levels                         │
│                                                                      │
│  Level          │ Durability                    │ Performance       │
│  ─────────────────────────────────────────────────────────────────  │
│  off            │ Commits immediately           │ Fastest           │
│                 │ (may lose recent commits)     │ (async)           │
│  ─────────────────────────────────────────────────────────────────  │
│  local          │ Local WAL flush only          │ Fast              │
│                 │ (no replication wait)         │                   │
│  ─────────────────────────────────────────────────────────────────  │
│  remote_write   │ Standby received WAL          │ Medium            │
│                 │ (not yet flushed to disk)     │                   │
│  ─────────────────────────────────────────────────────────────────  │
│  on             │ Standby flushed to disk       │ Slower            │
│                 │ (default, durable)            │                   │
│  ─────────────────────────────────────────────────────────────────  │
│  remote_apply   │ Standby replayed WAL          │ Slowest           │
│                 │ (visible on standby)          │                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Per-Transaction Control

```sql
-- Set synchronous commit per transaction
BEGIN;
SET LOCAL synchronous_commit = off;
-- Perform less critical updates
COMMIT;

-- Or for session
SET synchronous_commit = local;
```

---

## 6. Hot Standby

### Read Queries on Standby

```sql
-- Enable in postgresql.conf
hot_standby = on

-- Standby is read-only
SELECT * FROM users;  -- Works
INSERT INTO users ...; -- ERROR: cannot execute INSERT in a read-only transaction
```

### Handling Conflicts

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Hot Standby Conflicts                             │
│                                                                      │
│  Problem:                                                            │
│  Standby query reads row → Primary VACUUMs row → Conflict!          │
│                                                                      │
│  Solutions:                                                          │
│                                                                      │
│  1. hot_standby_feedback = on (standby sends feedback to primary)   │
│     Primary delays cleanup until standby is done                     │
│                                                                      │
│  2. max_standby_archive_delay / max_standby_streaming_delay         │
│     How long to wait before canceling conflicting query             │
│     Default: 30s. Set to -1 to never cancel                         │
│                                                                      │
│  3. vacuum_defer_cleanup_age (on primary)                           │
│     Delay cleanup by N transactions                                  │
└─────────────────────────────────────────────────────────────────────┘
```

```ini
# postgresql.conf on STANDBY

# Send feedback to primary about queries
hot_standby_feedback = on

# Wait time before canceling conflicting queries
max_standby_streaming_delay = 60s
max_standby_archive_delay = 60s
```

---

## 7. Cascading Replication

### Setup

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cascading Replication                             │
│                                                                      │
│  PRIMARY ──► STANDBY_1 ──► STANDBY_2                                │
│                       ──► STANDBY_3                                  │
│                                                                      │
│  Benefits:                                                           │
│  • Reduces load on primary                                           │
│  • Useful for geographically distributed standbys                   │
│  • STANDBY_2/3 can fail without affecting primary                   │
│                                                                      │
│  Considerations:                                                     │
│  • Additional replication lag                                        │
│  • STANDBY_1 must be running for cascade to work                    │
└─────────────────────────────────────────────────────────────────────┘
```

```ini
# postgresql.conf on STANDBY_1 (cascading source)
# Nothing special needed, just enable it
# Standby can be source for other standbys by default

# postgresql.conf on STANDBY_2 (cascade target)
primary_conninfo = 'host=standby1.example.com port=5432 user=replicator'
```

---

## 8. Monitoring Replication

### Replication Status Views

```sql
-- On PRIMARY: View connected standbys
SELECT
    client_addr,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    sync_state,
    sync_priority
FROM pg_stat_replication;

-- Replication lag in bytes
SELECT
    client_addr,
    pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS replay_lag_bytes,
    pg_wal_lsn_diff(pg_current_wal_lsn(), flush_lsn) AS flush_lag_bytes
FROM pg_stat_replication;

-- Lag in time (PostgreSQL 10+)
SELECT
    client_addr,
    write_lag,
    flush_lag,
    replay_lag
FROM pg_stat_replication;
```

### On Standby

```sql
-- WAL receiver status
SELECT
    status,
    receive_start_lsn,
    received_lsn,
    last_msg_send_time,
    last_msg_receipt_time
FROM pg_stat_wal_receiver;

-- Is this server in recovery?
SELECT pg_is_in_recovery();  -- true = standby

-- Last replayed LSN
SELECT pg_last_wal_receive_lsn();
SELECT pg_last_wal_replay_lsn();

-- Replay timestamp
SELECT pg_last_xact_replay_timestamp();
```

### Monitoring Queries

```sql
-- Comprehensive replication monitor
CREATE VIEW replication_status AS
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    state,
    sync_state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    pg_wal_lsn_diff(sent_lsn, replay_lsn) AS pending_bytes,
    write_lag,
    flush_lag,
    replay_lag
FROM pg_stat_replication;
```

---

## 9. Promoting Standby

### Manual Promotion

```bash
# Promote standby to primary

# Method 1: pg_ctl
pg_ctl promote -D /var/lib/postgresql/15/main

# Method 2: SQL function (PostgreSQL 12+)
SELECT pg_promote();

# Method 3: Trigger file (if configured)
touch /tmp/promote_trigger
```

### Post-Promotion Steps

```sql
-- Verify promotion
SELECT pg_is_in_recovery();  -- Should return false

-- Remove old primary connection
-- Edit postgresql.conf, remove or comment:
-- primary_conninfo = '...'

-- Allow writes
-- standby.signal file automatically removed
```

### Rebuilding Old Primary as Standby

```bash
# Option 1: pg_rewind (fast, uses WAL)
pg_rewind \
    --target-pgdata=/var/lib/postgresql/15/main \
    --source-server='host=new_primary port=5432 user=replicator' \
    --progress

# Option 2: Full pg_basebackup (slower but always works)
pg_basebackup -h new_primary -D /path/to/data -R
```

---

## 10. Configuration Summary

### Primary postgresql.conf

```ini
# Essential
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10

# Recommended
wal_keep_size = 1GB
hot_standby_feedback = on

# Optional: Synchronous
synchronous_standby_names = 'standby1'
synchronous_commit = on
```

### Standby postgresql.conf

```ini
# Essential
primary_conninfo = 'host=primary port=5432 user=replicator password=...'
hot_standby = on

# Recommended
primary_slot_name = 'standby1_slot'
hot_standby_feedback = on

# Conflict handling
max_standby_streaming_delay = 60s
```

---

## Summary

| Feature | Purpose |
|---------|---------|
| Streaming Replication | Real-time WAL streaming to standby |
| Hot Standby | Read queries on standby |
| Replication Slots | Guarantee WAL retention |
| Synchronous Replication | Durability guarantee |
| Cascading | Scale read replicas |

---

## Further Reading

- PostgreSQL Streaming Replication documentation
- pg_basebackup documentation
- High Availability and Load Balancing
