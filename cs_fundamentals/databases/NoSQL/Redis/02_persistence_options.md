# Redis Persistence Options

## Learning Objectives
- Understand RDB snapshots and AOF logging
- Configure persistence for different use cases
- Balance durability vs performance
- Implement backup and recovery strategies

---

## 1. Persistence Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Persistence Models                          │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    In-Memory Database                        │    │
│  │                          │                                   │    │
│  │             ┌────────────┴────────────┐                     │    │
│  │             │                         │                      │    │
│  │             ▼                         ▼                      │    │
│  │   ┌─────────────────┐       ┌─────────────────┐             │    │
│  │   │      RDB        │       │      AOF        │             │    │
│  │   │  (Snapshots)    │       │ (Append-Only)   │             │    │
│  │   │                 │       │                 │             │    │
│  │   │ Point-in-time   │       │ Write log of    │             │    │
│  │   │ binary dump     │       │ all operations  │             │    │
│  │   │                 │       │                 │             │    │
│  │   │ ✓ Compact       │       │ ✓ Durable       │             │    │
│  │   │ ✓ Fast load     │       │ ✓ Readable      │             │    │
│  │   │ ✗ Data loss     │       │ ✗ Larger files  │             │    │
│  │   │   between saves │       │ ✗ Slower writes │             │    │
│  │   └─────────────────┘       └─────────────────┘             │    │
│  │             │                         │                      │    │
│  │             └────────────┬────────────┘                     │    │
│  │                          ▼                                   │    │
│  │                   ┌─────────────┐                           │    │
│  │                   │    Disk     │                           │    │
│  │                   └─────────────┘                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Options:                                                            │
│  • RDB only    - Good for caching, backup                           │
│  • AOF only    - Maximum durability                                  │
│  • RDB + AOF   - Best of both worlds (recommended)                  │
│  • No persistence - Pure cache                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. RDB (Redis Database)

### How RDB Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RDB Snapshot Process                              │
│                                                                      │
│  1. Redis forks child process                                        │
│  2. Child writes memory to temp RDB file                            │
│  3. Temp file renamed to dump.rdb                                   │
│  4. Parent continues serving clients                                │
│                                                                      │
│  ┌─────────────┐         ┌─────────────┐                            │
│  │   Parent    │  fork   │   Child     │                            │
│  │  (serving)  │────────▶│ (writing)   │                            │
│  └─────────────┘         └──────┬──────┘                            │
│        │                        │                                    │
│        │                        ▼                                    │
│        │                 ┌─────────────┐                            │
│        │                 │ temp-xxx.rdb│                            │
│        │                 └──────┬──────┘                            │
│        │                        │ rename                             │
│        ▼                        ▼                                    │
│  ┌─────────────┐         ┌─────────────┐                            │
│  │   Memory    │         │  dump.rdb   │                            │
│  │ (Copy-on-   │         │ (complete)  │                            │
│  │   Write)    │         └─────────────┘                            │
│  └─────────────┘                                                     │
│                                                                      │
│  Note: Uses Copy-on-Write (COW) - pages copied only when modified   │
└─────────────────────────────────────────────────────────────────────┘
```

### Configuration

```bash
# redis.conf

# Save triggers: save <seconds> <changes>
save 900 1      # Save if 1 key changed in 900 seconds
save 300 10     # Save if 10 keys changed in 300 seconds
save 60 10000   # Save if 10000 keys changed in 60 seconds

# Disable RDB
save ""

# File settings
dbfilename dump.rdb
dir /var/lib/redis

# Compression (LZF)
rdbcompression yes

# Checksum
rdbchecksum yes

# Stop accepting writes if RDB fails
stop-writes-on-bgsave-error yes
```

### Manual Commands

```redis
# Foreground save (blocks!)
SAVE

# Background save (recommended)
BGSAVE

# Check last save time
LASTSAVE

# Check if save in progress
INFO persistence
# rdb_bgsave_in_progress:0
# rdb_last_save_time:1705344000
# rdb_last_bgsave_status:ok
```

### RDB Advantages and Disadvantages

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RDB Trade-offs                                    │
│                                                                      │
│  Advantages:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ ✓ Compact single file - easy to backup/transfer             │    │
│  │ ✓ Fast restart - just load binary file                      │    │
│  │ ✓ Good for disaster recovery                                │    │
│  │ ✓ Minimal impact on performance (fork + COW)                │    │
│  │ ✓ Allows setting different retention policies               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Disadvantages:                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ ✗ Data loss between snapshots (5 min = 5 min of data)       │    │
│  │ ✗ Fork can be slow with large datasets + slow disk          │    │
│  │ ✗ Fork uses memory (COW can double memory briefly)          │    │
│  │ ✗ Not suitable when minimal data loss required              │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. AOF (Append-Only File)

### How AOF Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AOF Write Process                                 │
│                                                                      │
│  Client         Redis           OS Buffer        Disk               │
│    │              │                 │              │                 │
│    │──SET foo──▶ │                 │              │                 │
│    │              │──*3\r\n...──▶  │              │                 │
│    │              │                 │              │                 │
│    │              │    ┌────fsync policy────┐     │                 │
│    │              │    │                    │     │                 │
│    │              │    │  always: every op  │     │                 │
│    │              │    │  everysec: 1/sec   │──▶ │                 │
│    │              │    │  no: OS decides    │     │                 │
│    │              │    │                    │     │                 │
│    │              │    └────────────────────┘     │                 │
│    │              │                               │                 │
│                                                                      │
│  AOF Format (RESP protocol):                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ *3                   # Array of 3 elements                   │    │
│  │ $3                   # 3-byte string follows                 │    │
│  │ SET                  # Command                               │    │
│  │ $3                   # 3-byte string follows                 │    │
│  │ foo                  # Key                                   │    │
│  │ $3                   # 3-byte string follows                 │    │
│  │ bar                  # Value                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Configuration

```bash
# redis.conf

# Enable AOF
appendonly yes

# File name
appendfilename "appendonly.aof"

# Fsync policy
appendfsync always     # Every write - safest, slowest
appendfsync everysec   # Every second - good balance (default)
appendfsync no         # OS decides - fastest, least safe

# Rewrite triggers
auto-aof-rewrite-percentage 100   # Rewrite when 100% larger
auto-aof-rewrite-min-size 64mb    # Minimum size to trigger

# Handle corrupted AOF
aof-load-truncated yes

# Use RDB preamble in AOF (faster loads)
aof-use-rdb-preamble yes
```

### AOF Rewrite

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AOF Rewrite Process                               │
│                                                                      │
│  Problem: AOF grows with every write                                │
│  SET counter 1                                                       │
│  INCR counter        (counter = 2)                                  │
│  INCR counter        (counter = 3)                                  │
│  INCR counter        (counter = 4)                                  │
│  ... thousands more...                                              │
│                                                                      │
│  Solution: Rewrite with current state only                          │
│  SET counter 1000    (one command instead of thousands)             │
│                                                                      │
│  Process:                                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 1. Fork child process                                       │    │
│  │ 2. Child writes new AOF from memory snapshot                │    │
│  │ 3. Parent buffers new writes                                │    │
│  │ 4. When child done, parent appends buffered writes          │    │
│  │ 5. Atomically rename new AOF to old                         │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

```redis
# Manual rewrite
BGREWRITEAOF

# Check status
INFO persistence
# aof_rewrite_in_progress:0
# aof_last_rewrite_time_sec:2
# aof_current_size:1234567
# aof_base_size:500000
```

### AOF Advantages and Disadvantages

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AOF Trade-offs                                    │
│                                                                      │
│  Advantages:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ ✓ More durable - lose at most 1 second of data              │    │
│  │ ✓ Append-only - no corruption on crash                      │    │
│  │ ✓ Human readable - can edit to fix issues                   │    │
│  │ ✓ Auto-rewrite keeps file size manageable                   │    │
│  │ ✓ Can recover from partial writes                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Disadvantages:                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ ✗ Larger files than RDB                                     │    │
│  │ ✗ Slower than RDB for same dataset                          │    │
│  │ ✗ Can have bugs with specific commands                      │    │
│  │ ✗ Fsync can cause latency spikes                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. RDB + AOF Combined

### Hybrid Persistence (Redis 4.0+)

```bash
# Enable RDB preamble in AOF file
aof-use-rdb-preamble yes

# Results in:
# [RDB format data][AOF format data]
# Fast load (RDB) + durability (AOF)
```

### Recovery Priority

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Startup Recovery Logic                            │
│                                                                      │
│  ┌─────────┐                                                         │
│  │  Start  │                                                         │
│  └────┬────┘                                                         │
│       │                                                              │
│       ▼                                                              │
│  ┌────────────────┐    Yes    ┌─────────────────┐                   │
│  │ AOF enabled?   │──────────▶│   Load AOF      │                   │
│  └────────┬───────┘           │ (more complete) │                   │
│           │ No                └─────────────────┘                   │
│           ▼                                                          │
│  ┌────────────────┐    Yes    ┌─────────────────┐                   │
│  │ RDB exists?    │──────────▶│   Load RDB      │                   │
│  └────────┬───────┘           └─────────────────┘                   │
│           │ No                                                       │
│           ▼                                                          │
│  ┌────────────────┐                                                  │
│  │  Empty start   │                                                  │
│  └────────────────┘                                                  │
│                                                                      │
│  Note: When both exist and AOF enabled, AOF takes precedence        │
└─────────────────────────────────────────────────────────────────────┘
```

### Recommended Configuration

```bash
# Production recommendation: RDB + AOF

# RDB for backups
save 900 1
save 300 10
save 60 10000
dbfilename dump.rdb

# AOF for durability
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-use-rdb-preamble yes
```

---

## 5. Fsync Policies

### Policy Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Fsync Policy Comparison                           │
│                                                                      │
│  ┌─────────────┬─────────────┬─────────────────────────────────┐    │
│  │   Policy    │  Data Loss  │         Performance             │    │
│  ├─────────────┼─────────────┼─────────────────────────────────┤    │
│  │   always    │   None      │ Slow (disk I/O every write)     │    │
│  │             │   (~0)      │ ~1000-10000 ops/sec             │    │
│  ├─────────────┼─────────────┼─────────────────────────────────┤    │
│  │   everysec  │   ~1 sec    │ Good (batch fsync)              │    │
│  │  (default)  │   of data   │ ~100,000+ ops/sec               │    │
│  ├─────────────┼─────────────┼─────────────────────────────────┤    │
│  │     no      │   ~30 sec   │ Fastest (OS decides)            │    │
│  │             │   of data   │ ~100,000+ ops/sec               │    │
│  └─────────────┴─────────────┴─────────────────────────────────┘    │
│                                                                      │
│  "everysec" provides best balance for most use cases                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Backup Strategies

### RDB Backup

```bash
# 1. Create backup from RDB file
cp /var/lib/redis/dump.rdb /backup/redis-$(date +%Y%m%d-%H%M%S).rdb

# 2. Or trigger save first
redis-cli BGSAVE
# Wait for completion
while [ $(redis-cli LASTSAVE) -eq $LASTSAVE ]; do sleep 1; done
cp /var/lib/redis/dump.rdb /backup/

# 3. Automated backup script
#!/bin/bash
BACKUP_DIR=/backup/redis
DATE=$(date +%Y%m%d-%H%M%S)
redis-cli BGSAVE
sleep 5  # Wait for save
cp /var/lib/redis/dump.rdb $BACKUP_DIR/dump-$DATE.rdb
# Keep last 7 days
find $BACKUP_DIR -name "dump-*.rdb" -mtime +7 -delete
```

### AOF Backup

```bash
# AOF files can be backed up while Redis is running
# Use cp, not mv (Redis holds file descriptor)
cp /var/lib/redis/appendonly.aof /backup/aof-$(date +%Y%m%d).aof

# For consistency, trigger rewrite first
redis-cli BGREWRITEAOF
# Wait for completion
while [ $(redis-cli INFO | grep aof_rewrite_in_progress | cut -d: -f2) -eq 1 ]; do
  sleep 1
done
cp /var/lib/redis/appendonly.aof /backup/
```

### Point-in-Time Recovery

```bash
# 1. Stop Redis
redis-cli SHUTDOWN NOSAVE

# 2. Replace data files
cp /backup/dump.rdb /var/lib/redis/dump.rdb
# or
cp /backup/appendonly.aof /var/lib/redis/appendonly.aof

# 3. Fix permissions
chown redis:redis /var/lib/redis/*

# 4. Start Redis
systemctl start redis
```

---

## 7. Recovery Procedures

### Corrupted AOF Recovery

```bash
# Check AOF file
redis-check-aof --fix /var/lib/redis/appendonly.aof

# Output:
# AOF analyzed: size=1234567, ok_up_to=1234500, diff=67
# This will shrink the AOF from 1234567 to 1234500 bytes
# Continue? [y/N]: y

# Manual edit (AOF is human-readable)
vim /var/lib/redis/appendonly.aof
# Find and fix/remove problematic commands
```

### Corrupted RDB Recovery

```bash
# Check RDB file
redis-check-rdb /var/lib/redis/dump.rdb

# If corrupted, restore from backup
cp /backup/dump-latest.rdb /var/lib/redis/dump.rdb
```

### Disaster Recovery

```bash
# Complete data loss scenario

# Option 1: Restore from RDB backup
redis-cli SHUTDOWN NOSAVE
cp /backup/dump.rdb /var/lib/redis/
chown redis:redis /var/lib/redis/dump.rdb
systemctl start redis

# Option 2: Restore from AOF backup
redis-cli SHUTDOWN NOSAVE
cp /backup/appendonly.aof /var/lib/redis/
# Disable RDB temporarily to ensure AOF is used
# redis.conf: appendonly yes
systemctl start redis

# Option 3: Restore from replica
# Promote replica to primary
redis-cli -h replica REPLICAOF NO ONE
```

---

## 8. Performance Considerations

### Fork Performance

```bash
# Check fork time
redis-cli INFO stats | grep latest_fork_usec
# latest_fork_usec:1234  # Microseconds

# Reduce fork impact:
# 1. Use faster storage (SSD/NVMe)
# 2. Limit dataset size
# 3. Disable THP (Transparent Huge Pages)
echo never > /sys/kernel/mm/transparent_hugepage/enabled

# 4. Reserve memory for fork
sysctl vm.overcommit_memory=1
```

### Monitoring Persistence

```redis
INFO persistence

# Key metrics:
# rdb_last_save_time: timestamp
# rdb_last_bgsave_status: ok/err
# rdb_last_bgsave_time_sec: duration
# aof_last_rewrite_time_sec: duration
# aof_current_size: bytes
# aof_buffer_length: pending writes
```

---

## 9. Configuration Recommendations

### Caching (No Persistence)

```bash
# Pure cache - no durability needed
save ""
appendonly no
```

### Standard Application

```bash
# Balance between performance and durability
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec
aof-use-rdb-preamble yes
```

### Maximum Durability

```bash
# Critical data - minimal data loss
save 60 1
appendonly yes
appendfsync always
```

### Large Dataset

```bash
# Minimize fork impact
save 3600 1           # Less frequent RDB
appendonly yes
appendfsync everysec
auto-aof-rewrite-min-size 512mb  # Larger before rewrite
```

---

## Summary

| Feature | RDB | AOF |
|---------|-----|-----|
| File Format | Binary | Text (RESP) |
| File Size | Compact | Larger |
| Durability | Minutes of data | Seconds of data |
| Startup Speed | Fast | Slower |
| CPU Usage | Periodic spike | Constant low |
| Use Case | Backup, cache | Primary data |

---

## Best Practices

```
✓ Use both RDB and AOF for production
✓ Set appendfsync to "everysec" for balance
✓ Enable aof-use-rdb-preamble for faster loads
✓ Monitor persistence metrics
✓ Test recovery procedures regularly
✓ Backup to remote storage
✓ Disable THP for consistent performance
✓ Size memory for fork overhead (2x in worst case)
```
