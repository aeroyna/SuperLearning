# Consistency Tuning

## Learning Objectives
- Understand Cassandra's consistency model
- Configure consistency levels for different use cases
- Balance availability, consistency, and performance
- Implement read/write path optimizations

---

## 1. CAP Theorem in Cassandra

### Cassandra's Trade-offs

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CAP Theorem                                       │
│                                                                      │
│                       Consistency                                    │
│                           ╱╲                                         │
│                          ╱  ╲                                        │
│                         ╱    ╲                                       │
│                        ╱ CP   ╲                                      │
│                       ╱        ╲                                     │
│                      ╱──────────╲                                    │
│       Availability ╱            ╲ Partition                          │
│                   ╱      AP      ╲ Tolerance                         │
│                  ╱                ╲                                  │
│                                                                      │
│  Cassandra: AP System (by default)                                  │
│  • Partition tolerant: handles network splits                       │
│  • Highly available: serves requests even during failures           │
│  • Tunable consistency: can choose consistency vs availability      │
│                                                                      │
│  Key Insight:                                                        │
│  Cassandra lets YOU choose the trade-off per operation!             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Consistency Levels

### Write Consistency Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Write Consistency Levels                          │
│                                                                      │
│  Level          │ Acknowledgments Required                          │
│  ────────────────────────────────────────────────────────────────── │
│  ANY            │ Any node (including hints)                        │
│  ONE            │ 1 replica                                         │
│  TWO            │ 2 replicas                                        │
│  THREE          │ 3 replicas                                        │
│  QUORUM         │ (RF/2) + 1 replicas (majority)                    │
│  LOCAL_QUORUM   │ Quorum in local datacenter                        │
│  EACH_QUORUM    │ Quorum in each datacenter                         │
│  ALL            │ All replicas                                      │
│  LOCAL_ONE      │ 1 replica in local datacenter                     │
│                                                                      │
│  Example: RF=3                                                       │
│  • QUORUM = (3/2)+1 = 2 replicas must acknowledge                   │
│  • ALL = 3 replicas must acknowledge                                │
│                                                                      │
│  Example: RF=3 per DC (2 DCs)                                       │
│  • LOCAL_QUORUM = 2 in local DC                                     │
│  • EACH_QUORUM = 2 in each DC (4 total)                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Read Consistency Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Read Consistency Levels                           │
│                                                                      │
│  Level          │ Replicas Contacted                                │
│  ────────────────────────────────────────────────────────────────── │
│  ONE            │ 1 replica (fastest, least consistent)             │
│  TWO            │ 2 replicas                                        │
│  THREE          │ 3 replicas                                        │
│  QUORUM         │ (RF/2) + 1 replicas                               │
│  LOCAL_QUORUM   │ Quorum in local datacenter                        │
│  EACH_QUORUM    │ Quorum in each datacenter                         │
│  ALL            │ All replicas (slowest, most consistent)           │
│  LOCAL_ONE      │ 1 replica in local datacenter                     │
│  SERIAL         │ For LWT, linearizable read                        │
│  LOCAL_SERIAL   │ SERIAL in local datacenter                        │
│                                                                      │
│  Read Repair:                                                        │
│  When multiple replicas return different data,                      │
│  coordinator repairs by sending latest to out-of-date replicas      │
└─────────────────────────────────────────────────────────────────────┘
```

### Achieving Strong Consistency

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Strong Consistency Formula                        │
│                                                                      │
│  Strong consistency when:                                            │
│  Write CL + Read CL > Replication Factor                            │
│                                                                      │
│  Examples with RF=3:                                                 │
│                                                                      │
│  QUORUM + QUORUM = 2 + 2 = 4 > 3 ✓ Strong                           │
│  ALL + ONE = 3 + 1 = 4 > 3 ✓ Strong                                 │
│  ONE + ALL = 1 + 3 = 4 > 3 ✓ Strong                                 │
│  ONE + ONE = 1 + 1 = 2 ≤ 3 ✗ Eventual                               │
│  QUORUM + ONE = 2 + 1 = 3 ≤ 3 ✗ Eventual                            │
│                                                                      │
│  Most common: QUORUM/QUORUM                                          │
│  • Good balance of consistency and availability                     │
│  • Tolerates up to (RF-1)/2 failed replicas                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Write Path

### Write Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cassandra Write Path                              │
│                                                                      │
│  Client ──────▶ Coordinator Node ──────▶ Replica Nodes             │
│                        │                       │                     │
│                        │                       ▼                     │
│                        │              1. Commit Log (append)         │
│                        │              2. MemTable (in-memory)        │
│                        │              3. ACK to coordinator          │
│                        │                       │                     │
│                        │◀──────────────────────┘                     │
│                        │                                             │
│                        ▼                                             │
│             Wait for CL acknowledgments                              │
│                        │                                             │
│                        ▼                                             │
│                  ACK to Client                                       │
│                                                                      │
│  Background (async):                                                 │
│  4. MemTable flush → SSTable (when full)                            │
│  5. Compaction (merge SSTables)                                     │
│                                                                      │
│  Key Point: Write is durable after commit log + memtable            │
│  No read-before-write = very fast writes                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Hinted Handoff

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Hinted Handoff                                    │
│                                                                      │
│  When replica is down:                                               │
│  1. Coordinator stores hint locally                                 │
│  2. Write succeeds if CL allows                                     │
│  3. When replica comes back, hints are delivered                    │
│                                                                      │
│  ┌─────────┐     Write     ┌─────────┐                              │
│  │  Client │──────────────▶│Coordinator│                             │
│  └─────────┘               └────┬────┘                              │
│                                 │                                    │
│               ┌─────────────────┼─────────────────┐                 │
│               ▼                 ▼                 ▼                 │
│         ┌─────────┐       ┌─────────┐       ┌─────────┐            │
│         │Replica 1│       │Replica 2│       │Replica 3│            │
│         │   ✓     │       │   ✓     │       │   ✗ DOWN│            │
│         └─────────┘       └─────────┘       └─────────┘            │
│                                                    │                 │
│                                              Hint stored            │
│                                              at coordinator         │
│                                                    │                 │
│                                              When back up:          │
│                                              Hint replayed          │
│                                                                      │
│  Configuration:                                                      │
│  max_hint_window_in_ms: 10800000  # 3 hours                         │
│  hinted_handoff_enabled: true                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Read Path

### Read Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cassandra Read Path                               │
│                                                                      │
│  Client ──────▶ Coordinator ──────▶ Replica(s)                      │
│                      │                   │                           │
│                      │                   ▼                           │
│                      │          1. Check MemTable                    │
│                      │          2. Check Row Cache (if enabled)      │
│                      │          3. Check Bloom Filter                │
│                      │          4. Check Partition Summary           │
│                      │          5. Check Partition Index             │
│                      │          6. Read SSTable(s)                   │
│                      │                   │                           │
│                      │◀──────────────────┘                           │
│                      │                                               │
│                      ▼                                               │
│           Merge results (if multiple replicas)                       │
│           Compare timestamps, return newest                          │
│                      │                                               │
│                      ▼                                               │
│             Read Repair (if inconsistent)                            │
│                      │                                               │
│                      ▼                                               │
│                Return to Client                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Read Repair

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Read Repair Types                                 │
│                                                                      │
│  1. Foreground Read Repair (blocking)                               │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ Triggered during read when CL > ONE                        │  │
│     │ Coordinator compares digests from replicas                 │  │
│     │ If mismatch: fetch full data, repair, then return         │  │
│     │ Adds latency but ensures consistency                      │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. Background Read Repair (async)                                  │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ Probabilistic: read_repair_chance                          │  │
│     │ Reads from all replicas in background                     │  │
│     │ Repairs if inconsistent                                   │  │
│     │ Doesn't affect read latency                               │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  Configuration:                                                      │
│  dclocal_read_repair_chance: 0.1  # 10% of reads                    │
│  read_repair_chance: 0.0          # Cross-DC (usually disabled)     │
└─────────────────────────────────────────────────────────────────────┘
```

### Speculative Retry

```yaml
# cassandra.yaml or per-table
speculative_retry: '99percentile'
# Options:
# - NONE
# - ALWAYS
# - 99percentile (99p of observed latency)
# - 50ms (fixed threshold)

# How it works:
# 1. Send read request to fastest replica
# 2. If response takes > threshold, send to next replica
# 3. Return first response received
# 4. Reduces tail latency
```

---

## 5. Consistency Patterns

### Eventually Consistent (Fast)

```python
# Use for: Analytics, logging, metrics
# Tolerates stale reads

cluster = Cluster()
session = cluster.connect('keyspace')

# Write with ONE
session.execute(
    "INSERT INTO events (id, data) VALUES (%s, %s)",
    [event_id, data],
    timeout=5.0,
    consistency_level=ConsistencyLevel.ONE
)

# Read with ONE
session.execute(
    "SELECT * FROM events WHERE id = %s",
    [event_id],
    consistency_level=ConsistencyLevel.ONE
)
```

### Strong Consistency

```python
# Use for: Financial transactions, inventory
# Requires quorum on both sides

from cassandra import ConsistencyLevel

# Write with QUORUM
session.execute(
    "UPDATE inventory SET count = count - 1 WHERE product_id = %s",
    [product_id],
    consistency_level=ConsistencyLevel.QUORUM
)

# Read with QUORUM
result = session.execute(
    "SELECT count FROM inventory WHERE product_id = %s",
    [product_id],
    consistency_level=ConsistencyLevel.QUORUM
)
```

### Local DC Priority

```python
# Use for: Multi-datacenter with local preference
# Reduces cross-DC latency

from cassandra.policies import DCAwareRoundRobinPolicy

cluster = Cluster(
    load_balancing_policy=DCAwareRoundRobinPolicy(local_dc='dc1')
)
session = cluster.connect('keyspace')

# Write to local DC
session.execute(
    "INSERT INTO users ...",
    consistency_level=ConsistencyLevel.LOCAL_QUORUM
)

# Read from local DC
session.execute(
    "SELECT * FROM users WHERE ...",
    consistency_level=ConsistencyLevel.LOCAL_ONE
)
```

---

## 6. Failure Scenarios

### Node Failure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Handling Node Failures                            │
│                                                                      │
│  RF=3, one node down:                                                │
│                                                                      │
│  Write CL=QUORUM: ✓ Works (needs 2/3, 2 available)                  │
│  Write CL=ALL:    ✗ Fails (needs 3/3, only 2 available)             │
│                                                                      │
│  Read CL=QUORUM:  ✓ Works (needs 2/3, 2 available)                  │
│  Read CL=ALL:     ✗ Fails (needs 3/3, only 2 available)             │
│                                                                      │
│  Best Practice:                                                      │
│  • Use QUORUM or LOCAL_QUORUM for fault tolerance                   │
│  • Avoid ALL in production                                          │
│  • Size cluster to handle node failures                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Datacenter Failure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Multi-DC Failure Handling                         │
│                                                                      │
│  DC1: RF=3, DC2: RF=3                                               │
│  DC2 becomes unavailable:                                           │
│                                                                      │
│  CL=LOCAL_QUORUM: ✓ DC1 continues operating                         │
│  CL=QUORUM:       Depends on total RF and available replicas        │
│  CL=EACH_QUORUM:  ✗ Fails (needs quorum in each DC)                 │
│                                                                      │
│  Best Practice:                                                      │
│  • Use LOCAL_QUORUM for DC isolation                                │
│  • Use QUORUM only when cross-DC consistency required               │
│  • Avoid EACH_QUORUM unless absolutely necessary                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Tuning for Performance

### Write-Heavy Workloads

```yaml
# cassandra.yaml

# Increase commit log size
commitlog_segment_size_in_mb: 64

# Tune memtable settings
memtable_heap_space_in_mb: 2048
memtable_offheap_space_in_mb: 2048
memtable_flush_writers: 4

# Concurrent writes
concurrent_writes: 64
```

```python
# Application level

# Use async writes
from cassandra.concurrent import execute_concurrent

statements = [(prepared, params) for params in batch_data]
execute_concurrent(session, statements, concurrency=100)

# Lower consistency for non-critical writes
session.execute(stmt, consistency_level=ConsistencyLevel.ONE)
```

### Read-Heavy Workloads

```yaml
# cassandra.yaml

# Enable row cache (for hot rows)
row_cache_size_in_mb: 1024

# Enable key cache
key_cache_size_in_mb: 512

# Tune concurrent reads
concurrent_reads: 32

# Speculative retry
speculative_retry: '99percentile'
```

```python
# Application level

# Use prepared statements
prepared = session.prepare("SELECT * FROM users WHERE id = ?")

# Enable tracing for slow queries
result = session.execute(stmt, trace=True)
print(result.get_query_trace())
```

---

## 8. Monitoring Consistency

### Key Metrics

```bash
# nodetool commands

# Check read/write latencies
nodetool tablehistograms keyspace table

# Check pending tasks
nodetool tpstats

# Check dropped messages
nodetool netstats
nodetool info

# Check hints
nodetool statushandoff
```

### JMX Metrics

```
# Key metrics to monitor:

# Latencies
org.apache.cassandra.metrics:type=ClientRequest,name=Latency

# Timeouts
org.apache.cassandra.metrics:type=ClientRequest,name=Timeouts

# Read/Write failures
org.apache.cassandra.metrics:type=ClientRequest,name=Failures

# Speculative retries
org.apache.cassandra.metrics:type=Table,name=SpeculativeRetries
```

---

## Summary

| CL | Write Durability | Read Consistency | Availability |
|----|------------------|------------------|--------------|
| ONE | Low | Low | High |
| QUORUM | Medium | Strong (with QUORUM read) | Medium |
| LOCAL_QUORUM | Medium | Strong in DC | High |
| ALL | High | Highest | Low |

---

## Best Practices

```
Consistency Levels:
✓ Use QUORUM/LOCAL_QUORUM for strong consistency
✓ Use LOCAL_* in multi-DC for latency
✓ Use ONE only for non-critical data
✗ Avoid ALL (no fault tolerance)

Write Path:
✓ Enable hinted handoff
✓ Use async writes for throughput
✓ Monitor commit log and memtable

Read Path:
✓ Enable speculative retry
✓ Use prepared statements
✓ Consider row cache for hot data

Multi-DC:
✓ Use LOCAL_QUORUM for DC isolation
✓ Configure DC-aware load balancing
✓ Plan for DC failure scenarios
```
