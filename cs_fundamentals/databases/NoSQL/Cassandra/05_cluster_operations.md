# Cluster Operations

## Learning Objectives
- Manage Cassandra cluster nodes
- Perform repairs and maintenance
- Handle compaction effectively
- Monitor cluster health and performance

---

## 1. Node Management

### Adding Nodes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Adding a Node                                     │
│                                                                      │
│  1. Configure new node                                              │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ cassandra.yaml:                                            │  │
│     │ cluster_name: 'MyCluster'                                  │  │
│     │ seeds: "node1,node2"                                       │  │
│     │ listen_address: <new_node_ip>                              │  │
│     │ rpc_address: <new_node_ip>                                 │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. Start the new node                                              │
│     $ cassandra                                                     │
│                                                                      │
│  3. Node bootstraps (streams data)                                  │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ • Joins cluster via seed nodes                             │  │
│     │ • Receives token assignment                                │  │
│     │ • Streams data for its token range                        │  │
│     │ • Marked as UP when complete                              │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  4. Run cleanup on other nodes (optional but recommended)           │
│     $ nodetool cleanup                                              │
│     # Removes data now owned by new node                           │
└─────────────────────────────────────────────────────────────────────┘
```

### Removing Nodes

```bash
# Graceful removal (node is UP)
# On the node to be removed:
nodetool decommission
# Streams data to other nodes, then leaves cluster

# Check progress
nodetool netstats

# Forced removal (node is DOWN)
# On any running node:
nodetool removenode <host_id>
# Get host_id from: nodetool status

# If node is unreachable for extended time
nodetool assassinate <ip_address>
# Last resort - may cause data loss
```

### Replacing a Dead Node

```bash
# 1. Note the dead node's address and tokens
nodetool status

# 2. Configure replacement node
# cassandra.yaml or environment:
# JVM_OPTS="$JVM_OPTS -Dcassandra.replace_address=<dead_node_ip>"

# 3. Start replacement node
cassandra

# 4. Remove JVM option after bootstrap completes

# 5. Run repair on replacement
nodetool repair -full
```

---

## 2. Repair Operations

### Why Repair?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Data Consistency Issues                           │
│                                                                      │
│  Without repair, data can become inconsistent:                       │
│                                                                      │
│  1. Failed writes during node outage                                │
│     Write → Node1 ✓, Node2 ✓, Node3 ✗ (down)                       │
│     Node3 never gets the write                                      │
│                                                                      │
│  2. Hinted handoff failure                                          │
│     Hints expire (default 3 hours)                                  │
│     Node returns after hints expired                                │
│                                                                      │
│  3. Partial consistency reads                                        │
│     CL=ONE may read stale data                                      │
│     No read repair triggered                                        │
│                                                                      │
│  4. Deleted data resurrection                                        │
│     Tombstones expired on some replicas                             │
│     Delete not seen by all replicas                                 │
│                                                                      │
│  Solution: Regular repairs sync all replicas                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Repair Types

```bash
# Full repair (compare all data)
nodetool repair -full keyspace table
# Use after node replacement or major outage

# Incremental repair (only unrepaired data)
nodetool repair keyspace table
# Default in Cassandra 4.0+, faster for regular maintenance

# Partitioner range repair
nodetool repair -pr keyspace table
# Only repairs data this node is primary for
# Use when running on all nodes

# Subrange repair (parallel repair)
nodetool repair -st <start_token> -et <end_token> keyspace table
# Repair specific token range

# Parallel repair (within node)
nodetool repair -par keyspace table
# Repairs multiple ranges simultaneously
```

### Repair Best Practices

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Repair Guidelines                                 │
│                                                                      │
│  Frequency:                                                          │
│  • Run within gc_grace_seconds (default 10 days)                    │
│  • Typically weekly for most clusters                               │
│  • More frequent for write-heavy workloads                          │
│                                                                      │
│  Scheduling:                                                         │
│  • Use repair scheduler (Cassandra Reaper, Datastax)                │
│  • Run during low-traffic periods                                   │
│  • Spread across cluster (don't repair all nodes at once)           │
│                                                                      │
│  Monitoring:                                                         │
│  • Watch pending compactions during repair                          │
│  • Monitor CPU and disk I/O                                         │
│  • Check for repair failures                                        │
│                                                                      │
│  Resource Management:                                                │
│  # Limit repair threads                                             │
│  nodetool repair -j 2 keyspace table                                │
│                                                                      │
│  # Throttle streaming                                                │
│  nodetool setstreamthroughput 200  # MB/s                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Using Cassandra Reaper

```yaml
# Reaper configuration (reaper-cassandra.yaml)
storageType: cassandra
cassandra:
  clusterName: "MyCluster"
  contactPoints: ["node1", "node2"]

# Create scheduled repair
curl -X POST "http://reaper:8080/repair_schedule" \
  -d "clusterName=MyCluster" \
  -d "keyspace=my_keyspace" \
  -d "scheduleDaysBetween=7" \
  -d "intensity=0.9"
```

---

## 3. Compaction

### How Compaction Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Compaction Process                                │
│                                                                      │
│  Write Path:                                                         │
│  MemTable → Flush → SSTable (immutable)                             │
│                                                                      │
│  Over time, multiple SSTables accumulate:                           │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                      │
│  │SST 1│ │SST 2│ │SST 3│ │SST 4│ │SST 5│                      │
│  │ key1│ │ key1│ │ key2│ │ key1│ │ key3│                      │
│  │  v1 │ │  v2 │ │  v1 │ │  v3 │ │  v1 │                      │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘                      │
│                                                                      │
│  Compaction merges and cleans up:                                    │
│  ┌────────────────────────────────────┐                             │
│  │          Merged SSTable            │                             │
│  │  key1: v3 (latest)                 │                             │
│  │  key2: v1                          │                             │
│  │  key3: v1                          │                             │
│  │  (tombstones removed if expired)   │                             │
│  └────────────────────────────────────┘                             │
│                                                                      │
│  Benefits:                                                           │
│  • Removes obsolete versions                                        │
│  • Removes expired tombstones                                       │
│  • Improves read performance                                        │
│  • Reclaims disk space                                              │
└─────────────────────────────────────────────────────────────────────┘
```

### Compaction Strategies

```sql
-- Size-Tiered Compaction (default)
CREATE TABLE data (
    id UUID PRIMARY KEY,
    value TEXT
) WITH compaction = {
    'class': 'SizeTieredCompactionStrategy',
    'min_threshold': 4,
    'max_threshold': 32
};
-- Good for: Write-heavy workloads
-- Pros: Low write amplification
-- Cons: Space amplification, read performance

-- Leveled Compaction
CREATE TABLE data (...)
WITH compaction = {
    'class': 'LeveledCompactionStrategy',
    'sstable_size_in_mb': 160
};
-- Good for: Read-heavy workloads
-- Pros: Predictable read performance, less space
-- Cons: Higher write amplification

-- Time-Window Compaction
CREATE TABLE time_series (...)
WITH compaction = {
    'class': 'TimeWindowCompactionStrategy',
    'compaction_window_unit': 'DAYS',
    'compaction_window_size': 1
};
-- Good for: Time-series with TTL
-- Pros: Efficient for time-based expiration
-- Cons: Only for time-series patterns
```

### Managing Compaction

```bash
# Check compaction status
nodetool compactionstats

# Check pending compactions
nodetool tpstats | grep Compaction

# Force major compaction (avoid in production!)
nodetool compact keyspace table

# Stop compaction
nodetool stop COMPACTION

# Set compaction throughput
nodetool setcompactionthroughput 64  # MB/s

# Disable compaction (for maintenance)
nodetool disableautocompaction keyspace table
nodetool enableautocompaction keyspace table
```

---

## 4. Backup and Restore

### Snapshot Backup

```bash
# Create snapshot
nodetool snapshot -t backup_2024_01_15 keyspace

# Snapshot location
ls /var/lib/cassandra/data/keyspace/table-uuid/snapshots/backup_2024_01_15/

# Clear snapshot (after backup copied)
nodetool clearsnapshot -t backup_2024_01_15 keyspace

# Automated backup script
#!/bin/bash
SNAPSHOT_NAME="backup_$(date +%Y%m%d_%H%M%S)"
nodetool snapshot -t $SNAPSHOT_NAME
# Copy snapshot to backup storage
aws s3 sync /var/lib/cassandra/data/keyspace/*/snapshots/$SNAPSHOT_NAME/ \
    s3://bucket/cassandra-backup/$SNAPSHOT_NAME/
nodetool clearsnapshot -t $SNAPSHOT_NAME
```

### Restore from Snapshot

```bash
# 1. Stop Cassandra
systemctl stop cassandra

# 2. Clear existing data
rm -rf /var/lib/cassandra/data/keyspace/table-*/

# 3. Copy snapshot files to table directory
cp /backup/snapshot_files/*.db /var/lib/cassandra/data/keyspace/table-uuid/

# 4. Update ownership
chown -R cassandra:cassandra /var/lib/cassandra/data

# 5. Start Cassandra
systemctl start cassandra

# 6. Refresh table (if Cassandra was running)
nodetool refresh keyspace table
```

### Incremental Backup

```yaml
# cassandra.yaml
incremental_backups: true
# Creates hard links in backups/ directory for each flushed SSTable

# Backup directory
/var/lib/cassandra/data/keyspace/table-uuid/backups/

# Combine with snapshots for point-in-time recovery
```

---

## 5. Monitoring

### nodetool Commands

```bash
# Cluster overview
nodetool status
nodetool ring
nodetool describecluster

# Node information
nodetool info
nodetool version
nodetool gossipinfo

# Performance
nodetool tpstats        # Thread pool stats
nodetool cfstats        # Table stats
nodetool tablehistograms keyspace table

# Cache stats
nodetool info | grep -i cache

# Garbage collection
nodetool gcstats

# Network
nodetool netstats
nodetool getsstables keyspace table key
```

### Key Metrics

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Critical Metrics to Monitor                       │
│                                                                      │
│  Cluster Health:                                                     │
│  • Node status (UN=Up Normal)                                       │
│  • Gossip convergence                                               │
│  • Pending tasks                                                    │
│  • Dropped messages                                                 │
│                                                                      │
│  Performance:                                                        │
│  • Read latency (p99, p999)                                         │
│  • Write latency (p99, p999)                                        │
│  • Pending compactions                                              │
│  • SSTable count per table                                          │
│                                                                      │
│  Resources:                                                          │
│  • Heap usage and GC pauses                                         │
│  • Disk space and I/O                                               │
│  • CPU utilization                                                  │
│  • Network bandwidth                                                │
│                                                                      │
│  Data:                                                               │
│  • Partition size (watch for large partitions)                      │
│  • Tombstone warnings                                               │
│  • Read/write failures                                              │
└─────────────────────────────────────────────────────────────────────┘
```

### JMX Monitoring

```python
# Using jolokia for JMX access
# Key beans to monitor:

# Client requests
org.apache.cassandra.metrics:type=ClientRequest,scope=Read,name=Latency
org.apache.cassandra.metrics:type=ClientRequest,scope=Write,name=Latency

# Compaction
org.apache.cassandra.metrics:type=Compaction,name=PendingTasks
org.apache.cassandra.metrics:type=Compaction,name=BytesCompacted

# Thread pools
org.apache.cassandra.metrics:type=ThreadPools,path=request,scope=*,name=ActiveTasks

# Storage
org.apache.cassandra.metrics:type=Storage,name=Load
org.apache.cassandra.metrics:type=Table,name=LiveDiskSpaceUsed
```

---

## 6. Performance Tuning

### JVM Settings

```bash
# cassandra-env.sh

# Heap size (typically 8-16GB, max 32GB)
MAX_HEAP_SIZE="16G"
HEAP_NEWSIZE="4G"

# G1GC settings (recommended for large heaps)
JVM_OPTS="$JVM_OPTS -XX:+UseG1GC"
JVM_OPTS="$JVM_OPTS -XX:G1RSetUpdatingPauseTimePercent=5"
JVM_OPTS="$JVM_OPTS -XX:MaxGCPauseMillis=500"

# GC logging
JVM_OPTS="$JVM_OPTS -Xlog:gc*:file=/var/log/cassandra/gc.log:time,uptime:filecount=10,filesize=100M"
```

### cassandra.yaml Tuning

```yaml
# Concurrency
concurrent_reads: 32
concurrent_writes: 32
concurrent_counter_writes: 32

# Memtable
memtable_heap_space_in_mb: 2048
memtable_offheap_space_in_mb: 2048
memtable_flush_writers: 4

# Compaction
concurrent_compactors: 4
compaction_throughput_mb_per_sec: 64

# Caches
key_cache_size_in_mb: 100
row_cache_size_in_mb: 0  # Usually leave disabled

# Timeouts
read_request_timeout_in_ms: 5000
write_request_timeout_in_ms: 2000
request_timeout_in_ms: 10000
```

### Disk Configuration

```bash
# Separate disks for data and commit log
data_file_directories:
    - /data1/cassandra/data
    - /data2/cassandra/data
commitlog_directory: /commitlog/cassandra/commitlog

# Use SSDs for best performance
# RAID 0 or JBOD for data directories
# Disable swap
swapoff -a

# Filesystem settings
# XFS recommended
# Mount with noatime,nodiratime
```

---

## 7. Troubleshooting

### Common Issues

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Troubleshooting Guide                             │
│                                                                      │
│  High Read Latency:                                                  │
│  • Check pending compactions                                        │
│  • Look for large partitions                                        │
│  • Check SSTable count                                              │
│  • Review consistency level                                         │
│  • Check row cache efficiency                                       │
│                                                                      │
│  High Write Latency:                                                 │
│  • Check commit log disk I/O                                        │
│  • Review memtable flushing                                         │
│  • Check hints queue                                                │
│  • Monitor thread pool pending tasks                                │
│                                                                      │
│  Node Down:                                                          │
│  • Check logs for errors                                            │
│  • Verify disk space                                                │
│  • Check GC pauses                                                  │
│  • Review gossip state                                              │
│                                                                      │
│  Tombstone Warnings:                                                 │
│  • Review data model (delete patterns)                              │
│  • Check gc_grace_seconds                                           │
│  • Run repair before gc_grace expires                               │
│  • Consider TTL instead of explicit deletes                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Diagnostic Commands

```bash
# Check system log
tail -f /var/log/cassandra/system.log

# Debug log
tail -f /var/log/cassandra/debug.log

# Query tracing
cqlsh> TRACING ON;
cqlsh> SELECT * FROM keyspace.table WHERE id = ?;

# Slow query log
nodetool setlogginglevel org.apache.cassandra.cql3 DEBUG

# Check for large partitions
nodetool tablehistograms keyspace table

# Check compaction history
nodetool compactionhistory
```

---

## Summary

| Operation | Command | When to Use |
|-----------|---------|-------------|
| Add node | Bootstrap | Scale out |
| Remove node | decommission | Scale down |
| Repair | nodetool repair | Weekly maintenance |
| Backup | nodetool snapshot | Before changes |
| Compact | nodetool compact | Rarely (mostly automatic) |

---

## Best Practices

```
Node Management:
✓ Add nodes one at a time
✓ Run cleanup after adding nodes
✓ Use decommission for graceful removal
✓ Replace dead nodes, don't leave gaps

Repair:
✓ Schedule regular repairs (weekly)
✓ Use repair scheduler (Reaper)
✓ Run within gc_grace_seconds
✓ Monitor repair progress and failures

Compaction:
✓ Choose strategy based on workload
✓ Monitor pending compactions
✓ Set appropriate throughput limits
✓ Avoid major compaction in production

Backup:
✓ Regular snapshots
✓ Test restore procedures
✓ Store backups off-cluster
✓ Consider incremental backups

Monitoring:
✓ Track latency percentiles (p99, p999)
✓ Alert on pending compactions
✓ Monitor heap and GC
✓ Watch for large partitions
```
