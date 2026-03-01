# Partitioning Strategy

## Learning Objectives
- Choose effective partition keys
- Understand data distribution mechanisms
- Avoid hotspots and unbalanced clusters
- Size partitions appropriately

---

## 1. Partitioning Fundamentals

### How Partitioning Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Data Distribution                                 │
│                                                                      │
│  Step 1: Hash partition key                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  partition_key = "user123"                                  │    │
│  │  token = hash(partition_key) = -2^63 to 2^63                │    │
│  │  token = -5234567890123456789                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Step 2: Map token to node via token ring                           │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │              -2^63 ─────────────────── 2^63                  │    │
│  │                                                              │    │
│  │   Node A        Node B        Node C        Node D          │    │
│  │   -2^63         -2^62         0             2^62            │    │
│  │   to            to            to            to              │    │
│  │   -2^62         0             2^62          2^63            │    │
│  │                                                              │    │
│  │   token -5234... falls in Node A's range                    │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Step 3: Replicate to N nodes (replication_factor = 3)             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Primary: Node A                                            │    │
│  │  Replica 1: Node B (next in ring)                           │    │
│  │  Replica 2: Node C (next in ring)                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Partitioners

```yaml
# cassandra.yaml

# Murmur3Partitioner (default, recommended)
partitioner: org.apache.cassandra.dht.Murmur3Partitioner
# - Good distribution
# - Fast hashing
# - 64-bit token range

# RandomPartitioner (legacy)
partitioner: org.apache.cassandra.dht.RandomPartitioner
# - MD5 hash
# - 128-bit tokens
# - Older compatibility

# ByteOrderedPartitioner (rarely used)
partitioner: org.apache.cassandra.dht.ByteOrderedPartitioner
# - Preserves byte order
# - Enables range scans
# - Easy to create hotspots
```

---

## 2. Partition Key Selection

### Good Partition Keys

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Partition Key Criteria                            │
│                                                                      │
│  ✓ High Cardinality                                                  │
│    Many unique values = even distribution                           │
│    Good: user_id, order_id, device_id                               │
│    Bad: country (only ~200 values)                                  │
│                                                                      │
│  ✓ Query Access Pattern                                              │
│    Partition key must be in every query                             │
│    Choose based on how data is accessed                             │
│                                                                      │
│  ✓ Even Write Distribution                                           │
│    Avoid keys that concentrate writes                               │
│    Bad: date (all today's writes to one partition)                  │
│                                                                      │
│  ✓ Bounded Partition Size                                            │
│    Partition should not grow unbounded                              │
│    Use time buckets or composite keys                               │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples

```sql
-- Good: User-centric data
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY,
    name TEXT,
    email TEXT
);
-- Each user = one partition, evenly distributed

-- Good: Time-series with bucketing
CREATE TABLE sensor_readings (
    sensor_id TEXT,
    date DATE,
    timestamp TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((sensor_id, date), timestamp)
);
-- Partition per sensor per day, bounded size

-- Bad: Low cardinality
CREATE TABLE users_by_country (
    country TEXT PRIMARY KEY,
    user_id UUID,
    name TEXT
);
-- Only ~200 partitions, some huge (US, India, China)

-- Bad: Monotonically increasing
CREATE TABLE events_by_time (
    event_date DATE PRIMARY KEY,
    event_id UUID,
    data TEXT
);
-- Today's date = hot partition getting all writes
```

---

## 3. Composite Partition Keys

### When to Use

```sql
-- Composite partition key: ((col1, col2), clustering...)
-- All columns together determine partition

-- Time-series: bucket by time
CREATE TABLE metrics (
    host_id TEXT,
    metric_date DATE,
    metric_time TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((host_id, metric_date), metric_time)
);
-- One partition per host per day

-- Multi-tenant: include tenant
CREATE TABLE tenant_data (
    tenant_id UUID,
    entity_id UUID,
    data TEXT,
    PRIMARY KEY ((tenant_id, entity_id))
);
-- Each tenant-entity combination is one partition

-- Geographic: region + entity
CREATE TABLE regional_data (
    region TEXT,
    data_id UUID,
    created_at TIMESTAMP,
    PRIMARY KEY ((region, data_id))
);
```

### Query Implications

```sql
-- Composite partition key = ALL parts required in query

-- This works:
SELECT * FROM metrics
WHERE host_id = 'server1' AND metric_date = '2024-01-15';

-- This does NOT work:
SELECT * FROM metrics WHERE host_id = 'server1';
-- Error: Missing partition key column metric_date

-- Workaround: Query multiple partitions
SELECT * FROM metrics
WHERE host_id = 'server1'
AND metric_date IN ('2024-01-15', '2024-01-14', '2024-01-13');
-- Queries 3 partitions
```

---

## 4. Avoiding Hotspots

### Hotspot Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Common Hotspot Causes                             │
│                                                                      │
│  1. Time-based keys                                                  │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ Date as partition key → all writes to "today"             │  │
│     │ Timestamp → monotonic, all to latest partition            │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  2. Sequential IDs                                                   │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ Auto-increment ID → clustered on one node                 │  │
│     │ Use UUID or TIMEUUID instead                              │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  3. Popular entities                                                 │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ Celebrity user partition gets millions of reads/writes    │  │
│     │ Consider sharding hot partitions                          │  │
│     └────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  4. Low cardinality                                                  │
│     ┌────────────────────────────────────────────────────────────┐  │
│     │ status = 'active' → millions of rows, few partitions      │  │
│     │ Add additional partition key column                       │  │
│     └────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### Solutions

```sql
-- Problem: All today's events to one partition
CREATE TABLE events_bad (
    event_date DATE,
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY (event_date, event_id)
);

-- Solution 1: Add random bucket
CREATE TABLE events_bucketed (
    event_date DATE,
    bucket INT,  -- 0-99
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY ((event_date, bucket), event_id)
);
-- Insert: bucket = random(0, 100)
-- Read: query all 100 buckets, merge results

-- Solution 2: Include source/region
CREATE TABLE events_by_source (
    event_date DATE,
    source_id TEXT,  -- Many sources
    event_id TIMEUUID,
    data TEXT,
    PRIMARY KEY ((event_date, source_id), event_id)
);

-- Problem: Celebrity user partition too hot
-- Solution: Shard the user
CREATE TABLE user_followers_sharded (
    user_id UUID,
    shard INT,  -- 0-15
    follower_id UUID,
    followed_at TIMESTAMP,
    PRIMARY KEY ((user_id, shard), followed_at, follower_id)
);
-- Insert: shard = hash(follower_id) % 16
-- Count: query all 16 shards, sum results
```

---

## 5. Partition Sizing

### Size Limits

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Partition Size Guidelines                         │
│                                                                      │
│  Recommended Limits:                                                 │
│  • < 100 MB per partition                                           │
│  • < 100,000 rows per partition                                     │
│  • Absolute max: 2 billion cells                                    │
│                                                                      │
│  Why These Limits?                                                   │
│  • Large partitions = slow reads                                    │
│  • Memory pressure during compaction                                │
│  • Repair takes longer                                              │
│  • Streaming during node operations                                 │
│                                                                      │
│  Symptoms of Large Partitions:                                       │
│  • Slow queries on specific keys                                    │
│  • GC pauses                                                        │
│  • Compaction failures                                              │
│  • Timeouts                                                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Calculating Partition Size

```python
# Estimate partition size

# Row size components:
# - Partition key: size of partition key columns
# - Clustering key: size of clustering columns
# - Regular columns: sum of all column sizes
# - Overhead: ~20-30 bytes per row

# Example: Sensor readings
# - partition: sensor_id (8 bytes) + date (4 bytes)
# - clustering: timestamp (8 bytes)
# - data: value (8 bytes), status (1 byte)
# - overhead: ~25 bytes

row_size = 8 + 4 + 8 + 8 + 1 + 25  # ~54 bytes

# Readings per day (1 per second)
rows_per_partition = 86400

partition_size = row_size * rows_per_partition
# 54 * 86,400 = 4.6 MB per partition ✓

# If 1 reading per millisecond:
rows_per_partition = 86400000
partition_size = 54 * 86400000
# = 4.6 GB per partition ✗ Too large!
# Solution: bucket by hour instead of day
```

### Bucketing Strategies

```sql
-- Time-based bucketing examples

-- By day (medium volume)
PRIMARY KEY ((sensor_id, date), timestamp)

-- By hour (high volume)
PRIMARY KEY ((sensor_id, date, hour), timestamp)

-- By minute (very high volume)
PRIMARY KEY ((sensor_id, bucket_minute), timestamp)
-- bucket_minute = floor(timestamp / 60000)

-- By count (fixed partition size)
PRIMARY KEY ((entity_id, bucket), sequence)
-- New bucket every N rows
-- Requires application logic to manage bucket numbers
```

---

## 6. Token Awareness

### Understanding Tokens

```sql
-- Get token for a key
SELECT token(user_id), user_id FROM users;

-- Token function in queries
SELECT * FROM users
WHERE token(user_id) > -9223372036854775808
AND token(user_id) <= -4611686018427387904;
-- Query range of tokens (for parallel processing)

-- Useful for:
-- - Splitting large queries
-- - Parallel processing
-- - Understanding data distribution
```

### Token-Aware Clients

```python
from cassandra.cluster import Cluster
from cassandra.policies import TokenAwarePolicy, RoundRobinPolicy

# Token-aware routing
cluster = Cluster(
    ['node1', 'node2', 'node3'],
    load_balancing_policy=TokenAwarePolicy(RoundRobinPolicy())
)

# Driver routes queries directly to replica nodes
# Reduces network hops for better performance
```

---

## 7. Monitoring Distribution

### Checking Partition Distribution

```bash
# nodetool commands

# Check data distribution across nodes
nodetool status

# Check table statistics
nodetool tablestats keyspace.table

# Check estimated partition count
nodetool tablehistograms keyspace.table

# Find large partitions
nodetool toppartitions keyspace table 5000 10
# Samples for 5000ms, shows top 10 partitions
```

### What to Monitor

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Distribution Metrics                              │
│                                                                      │
│  nodetool status output:                                             │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ Datacenter: dc1                                                │ │
│  │ ===============                                                │ │
│  │ Status=Up/Down  Load     Tokens  Owns (effective)              │ │
│  │ UN  node1      100.5 GB  256     33.3%                         │ │
│  │ UN  node2      98.2 GB   256     33.1%                         │ │
│  │ UN  node3      101.1 GB  256     33.6%                         │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  Look for:                                                           │
│  • Load should be roughly equal across nodes                        │
│  • Large differences indicate partition key issues                  │
│  • Owns % should be even                                            │
│                                                                      │
│  tablehistograms output:                                             │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ Partition Size (bytes):                                        │ │
│  │   Min: 100  Max: 50000000  Mean: 100000                        │ │
│  │   50%: 50000  75%: 100000  95%: 500000  99%: 5000000           │ │
│  └────────────────────────────────────────────────────────────────┘ │
│  Watch for: Max >> Mean (indicates large partitions)               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Replication Strategies

### SimpleStrategy

```sql
-- For development/single datacenter
CREATE KEYSPACE test_ks
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 3
};

-- Places replicas on next N-1 nodes in ring
-- Does not consider datacenter topology
```

### NetworkTopologyStrategy

```sql
-- For production/multi-datacenter
CREATE KEYSPACE production_ks
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3,
    'dc2': 2
};

-- Replicas distributed across racks within each DC
-- Survives rack and datacenter failures
```

### Rack Awareness

```yaml
# cassandra-rackdc.properties
dc=dc1
rack=rack1

# Cassandra places replicas on different racks when possible
# Survives rack-level failures
```

---

## Summary

| Factor | Recommendation |
|--------|----------------|
| Cardinality | High (millions of unique values) |
| Distribution | Even across nodes |
| Partition Size | < 100MB, < 100K rows |
| Write Pattern | No hot partitions |
| Query Pattern | Partition key in every query |

---

## Best Practices

```
Partition Key Selection:
✓ High cardinality
✓ Required in all queries
✓ Evenly distributed writes
✓ Bounded growth

Sizing:
✓ Calculate expected partition size
✓ Use time bucketing for time-series
✓ Monitor partition size metrics
✓ Shard hot partitions

Distribution:
✓ Use NetworkTopologyStrategy in production
✓ Enable token-aware clients
✓ Monitor load balance across nodes
✓ Consider rack awareness

Avoid:
✗ Date/time as sole partition key
✗ Low cardinality keys
✗ Unbounded partitions
✗ Sequential/monotonic keys
```
