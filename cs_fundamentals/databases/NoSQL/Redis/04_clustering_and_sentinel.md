# Clustering and Sentinel

## Learning Objectives
- Configure Redis Sentinel for high availability
- Deploy Redis Cluster for horizontal scaling
- Understand failover mechanisms
- Design resilient Redis architectures

---

## 1. Replication Fundamentals

### Master-Replica Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Replication                                 │
│                                                                      │
│                      ┌─────────────────┐                            │
│                      │     Master      │                            │
│                      │   (Read/Write)  │                            │
│                      │   Port: 6379    │                            │
│                      └────────┬────────┘                            │
│                               │                                      │
│            ┌──────────────────┼──────────────────┐                  │
│            │                  │                  │                   │
│            ▼                  ▼                  ▼                   │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐       │
│  │    Replica 1    │ │    Replica 2    │ │    Replica 3    │       │
│  │   (Read Only)   │ │   (Read Only)   │ │   (Read Only)   │       │
│  │   Port: 6380    │ │   Port: 6381    │ │   Port: 6382    │       │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘       │
│                                                                      │
│  Replication Flow:                                                   │
│  1. Replica connects to master                                      │
│  2. Master sends RDB snapshot                                       │
│  3. Master streams write commands continuously                      │
│  4. Replica applies commands                                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Configuration

```bash
# redis.conf for master
bind 0.0.0.0
port 6379

# redis.conf for replica
bind 0.0.0.0
port 6380
replicaof 192.168.1.100 6379     # Master address
replica-read-only yes            # Default: yes
replica-serve-stale-data yes     # Serve data during sync
```

```redis
# Runtime commands
REPLICAOF 192.168.1.100 6379    # Become replica
REPLICAOF NO ONE                 # Promote to master

# Check replication status
INFO replication
# role:master
# connected_slaves:3
# slave0:ip=192.168.1.101,port=6380,state=online

# On replica
INFO replication
# role:slave
# master_host:192.168.1.100
# master_port:6379
# master_link_status:up
```

### Replication Internals

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Sync Process                                      │
│                                                                      │
│  Full Sync (Initial):                                                │
│  1. Replica: PSYNC ? -1                                             │
│  2. Master: FULLRESYNC <replid> <offset>                            │
│  3. Master: Send RDB + buffered commands                            │
│  4. Replica: Load RDB + apply commands                              │
│                                                                      │
│  Partial Sync (Reconnection):                                        │
│  1. Replica: PSYNC <replid> <offset>                                │
│  2. Master: Check backlog buffer                                    │
│  3. If offset in buffer: CONTINUE + missing commands                │
│  4. If not: Full sync                                               │
│                                                                      │
│  Backlog Buffer:                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ [cmd1][cmd2][cmd3]...[cmdN]                                  │   │
│  │ offset: 1000  ─────────────────────────▶  offset: 5000      │   │
│  └──────────────────────────────────────────────────────────────┘   │
│  repl-backlog-size 1mb (default)                                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Redis Sentinel

### Sentinel Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Sentinel Setup                              │
│                                                                      │
│          ┌─────────────────────────────────────────────┐            │
│          │            Sentinel Cluster                 │            │
│          │                                             │            │
│          │  ┌────────┐  ┌────────┐  ┌────────┐        │            │
│          │  │Sentinel│  │Sentinel│  │Sentinel│        │            │
│          │  │  :26379│  │  :26380│  │  :26381│        │            │
│          │  └────┬───┘  └────┬───┘  └────┬───┘        │            │
│          │       │           │           │             │            │
│          │       └───────────┼───────────┘             │            │
│          │                   │                         │            │
│          └───────────────────┼─────────────────────────┘            │
│                              │                                       │
│                    ┌─────────┴─────────┐                            │
│                    │                   │                             │
│              ┌─────▼─────┐       ┌─────▼─────┐                      │
│              │  Master   │       │  Replica  │                      │
│              │   :6379   │◀─────▶│   :6380   │                      │
│              └───────────┘       └───────────┘                      │
│                                                                      │
│  Sentinel Responsibilities:                                          │
│  • Monitor master and replicas                                      │
│  • Notify clients on state changes                                  │
│  • Automatic failover when master fails                             │
│  • Configuration provider for clients                               │
└─────────────────────────────────────────────────────────────────────┘
```

### Sentinel Configuration

```bash
# sentinel.conf
port 26379
sentinel monitor mymaster 192.168.1.100 6379 2
# 2 = quorum (sentinels needed to agree on failure)

sentinel auth-pass mymaster yourpassword
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 60000

# Notification scripts
sentinel notification-script mymaster /path/to/notify.sh
sentinel client-reconfig-script mymaster /path/to/reconfig.sh
```

```bash
# Start sentinel
redis-sentinel /path/to/sentinel.conf
# or
redis-server /path/to/sentinel.conf --sentinel
```

### Sentinel Commands

```redis
# Connect to sentinel
redis-cli -p 26379

# Get master address
SENTINEL get-master-addr-by-name mymaster
# 1) "192.168.1.100"
# 2) "6379"

# List masters
SENTINEL masters

# List replicas
SENTINEL replicas mymaster

# List sentinels
SENTINEL sentinels mymaster

# Check master state
SENTINEL ckquorum mymaster
# OK 3 usable Sentinels. Quorum and failover authorization is possible

# Force failover
SENTINEL failover mymaster

# Reset sentinel state
SENTINEL reset mymaster
```

### Failover Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Failover Sequence                                 │
│                                                                      │
│  1. Detection (SDOWN → ODOWN)                                       │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ Sentinel 1: Master not responding for 5000ms              │   │
│     │ Sentinel 1 marks master as SDOWN (Subjectively Down)     │   │
│     │                                                          │   │
│     │ Sentinel 1 asks other sentinels                          │   │
│     │ If quorum agrees: ODOWN (Objectively Down)               │   │
│     └──────────────────────────────────────────────────────────┘   │
│                                                                      │
│  2. Leader Election                                                  │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ Sentinels vote for one to perform failover               │   │
│     │ Uses Raft-like consensus algorithm                       │   │
│     └──────────────────────────────────────────────────────────┘   │
│                                                                      │
│  3. Replica Selection                                                │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ Priority: replica-priority (lower = preferred)           │   │
│     │ Offset: Most up-to-date replica                          │   │
│     │ Run ID: Lexicographically smallest                       │   │
│     └──────────────────────────────────────────────────────────┘   │
│                                                                      │
│  4. Promotion                                                        │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ Selected replica: REPLICAOF NO ONE                       │   │
│     │ Other replicas: REPLICAOF new-master                     │   │
│     │ Old master (if returns): Becomes replica                 │   │
│     └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Client Connection

```python
from redis.sentinel import Sentinel

# Connect to sentinels
sentinel = Sentinel([
    ('sentinel1.example.com', 26379),
    ('sentinel2.example.com', 26379),
    ('sentinel3.example.com', 26379)
], socket_timeout=0.1)

# Get master connection (follows failover)
master = sentinel.master_for('mymaster', socket_timeout=0.1)
master.set('foo', 'bar')

# Get replica for reads
slave = sentinel.slave_for('mymaster', socket_timeout=0.1)
slave.get('foo')

# Connection pool auto-reconnects on failover
```

---

## 3. Redis Cluster

### Cluster Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Cluster                                     │
│                                                                      │
│  16384 Hash Slots distributed across nodes                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 0────────5460│5461───────10922│10923──────16383             │    │
│  │    Slots    │     Slots      │     Slots                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│         │               │               │                            │
│         ▼               ▼               ▼                            │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                    │
│  │   Node A    │ │   Node B    │ │   Node C    │                    │
│  │   Master    │ │   Master    │ │   Master    │                    │
│  │  :7000      │ │  :7001      │ │  :7002      │                    │
│  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘                    │
│         │               │               │                            │
│  ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐                    │
│  │   Node A'   │ │   Node B'   │ │   Node C'   │                    │
│  │   Replica   │ │   Replica   │ │   Replica   │                    │
│  │  :7003      │ │  :7004      │ │  :7005      │                    │
│  └─────────────┘ └─────────────┘ └─────────────┘                    │
│                                                                      │
│  Key → Slot mapping: CRC16(key) mod 16384                           │
│  Hash tags: {user}:profile and {user}:settings → same slot          │
└─────────────────────────────────────────────────────────────────────┘
```

### Cluster Setup

```bash
# Create cluster nodes
# Node configuration
port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000
appendonly yes

# Start all nodes
redis-server redis-7000.conf
redis-server redis-7001.conf
# ... etc

# Create cluster
redis-cli --cluster create \
  192.168.1.100:7000 192.168.1.101:7001 192.168.1.102:7002 \
  192.168.1.103:7003 192.168.1.104:7004 192.168.1.105:7005 \
  --cluster-replicas 1
# --cluster-replicas 1 means 1 replica per master
```

### Cluster Commands

```redis
# Connect to cluster
redis-cli -c -p 7000  # -c enables cluster mode

# Cluster info
CLUSTER INFO
# cluster_state:ok
# cluster_slots_assigned:16384
# cluster_known_nodes:6

# Node list
CLUSTER NODES
# <id> <ip:port> <flags> <master-id> <ping-sent> <pong-recv> <config-epoch> <link-state> <slot range>

# Slot info
CLUSTER SLOTS
CLUSTER KEYSLOT mykey  # Which slot for key

# Add node
CLUSTER MEET 192.168.1.106 7006

# Reshard slots
redis-cli --cluster reshard 192.168.1.100:7000
# Interactive: specify slots to move and target node

# Check cluster health
redis-cli --cluster check 192.168.1.100:7000

# Fix cluster issues
redis-cli --cluster fix 192.168.1.100:7000
```

### Hash Tags

```redis
# Keys with same hash tag go to same slot
# Hash tag: content between first { and first }

# These go to same slot (tag = "user123")
SET {user123}:profile "..."
SET {user123}:settings "..."
SET {user123}:cache "..."

# Enables multi-key operations
MGET {user123}:profile {user123}:settings

# Without hash tags - may fail with CROSSSLOT error
MGET user:123:profile user:456:profile
# Error: CROSSSLOT Keys in request don't hash to same slot
```

### Cluster Client (Python)

```python
from redis.cluster import RedisCluster

# Connect to cluster
rc = RedisCluster(
    host='192.168.1.100',
    port=7000,
    # OR
    # startup_nodes=[
    #     {'host': '192.168.1.100', 'port': 7000},
    #     {'host': '192.168.1.101', 'port': 7001},
    # ]
)

# Automatic routing
rc.set('foo', 'bar')  # Routed to correct node
rc.get('foo')

# Multi-key with hash tags
rc.mset({'{user}:name': 'John', '{user}:email': 'john@example.com'})
rc.mget('{user}:name', '{user}:email')

# Pipeline (within same slot)
pipe = rc.pipeline()
pipe.set('{user}:a', '1')
pipe.set('{user}:b', '2')
pipe.execute()
```

---

## 4. Cluster Operations

### Adding Nodes

```bash
# Add new master
redis-cli --cluster add-node 192.168.1.106:7006 192.168.1.100:7000

# Add as replica of specific master
redis-cli --cluster add-node 192.168.1.107:7007 192.168.1.100:7000 \
  --cluster-slave --cluster-master-id <master-node-id>

# Reshard slots to new node
redis-cli --cluster reshard 192.168.1.100:7000 \
  --cluster-from all \
  --cluster-to <new-node-id> \
  --cluster-slots 4096
```

### Removing Nodes

```bash
# First reshard slots away from node
redis-cli --cluster reshard 192.168.1.100:7000 \
  --cluster-from <node-id-to-remove> \
  --cluster-to <other-node-id> \
  --cluster-slots 5461

# Then delete empty node
redis-cli --cluster del-node 192.168.1.100:7000 <node-id-to-remove>
```

### Failover

```redis
# On replica, trigger manual failover
CLUSTER FAILOVER

# Force failover (even if master unreachable)
CLUSTER FAILOVER FORCE

# Takeover (no master agreement needed)
CLUSTER FAILOVER TAKEOVER
```

### Rebalancing

```bash
# Automatically rebalance slots
redis-cli --cluster rebalance 192.168.1.100:7000

# With weights (relative slot distribution)
redis-cli --cluster rebalance 192.168.1.100:7000 \
  --cluster-weight <node-id>=2 <node-id>=1 <node-id>=1

# Simulate only
redis-cli --cluster rebalance 192.168.1.100:7000 --cluster-simulate
```

---

## 5. Cluster Limitations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cluster Limitations                               │
│                                                                      │
│  Multi-Key Operations:                                               │
│  ✗ MGET, MSET across slots (without hash tags)                      │
│  ✗ SUNION, SINTER across slots                                      │
│  ✗ Transactions (MULTI/EXEC) across slots                          │
│  ✗ Lua scripts accessing multiple slots                             │
│                                                                      │
│  Database Selection:                                                 │
│  ✗ Only database 0 supported                                        │
│  ✗ SELECT command not available                                     │
│                                                                      │
│  Pub/Sub:                                                            │
│  ✓ Works, but messages broadcast to all nodes                       │
│  ⚠ Higher network overhead                                          │
│                                                                      │
│  Workarounds:                                                        │
│  • Use hash tags for related keys                                   │
│  • Design data model around single-slot operations                  │
│  • Use Lua scripts within single slot                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Sentinel vs Cluster

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Comparison                                        │
│                                                                      │
│  Feature              │ Sentinel        │ Cluster                   │
│  ─────────────────────┼─────────────────┼──────────────────────────│
│  Purpose              │ High Availability│ HA + Horizontal Scale   │
│  Data Partitioning    │ No              │ Yes (sharding)           │
│  Max Data Size        │ Single node RAM │ Aggregate of all nodes   │
│  Write Scaling        │ No              │ Yes                      │
│  Multi-key Ops        │ Full support    │ Same-slot only           │
│  Transactions         │ Full support    │ Same-slot only           │
│  Complexity           │ Lower           │ Higher                   │
│  Min Nodes            │ 3 sentinels     │ 6 (3 masters+3 replicas) │
│                       │ + 1 master      │                          │
│                       │ + 1 replica     │                          │
│                                                                      │
│  Choose Sentinel when:                                               │
│  • Data fits in single node                                         │
│  • Need simple HA                                                   │
│  • Use multi-key operations extensively                             │
│                                                                      │
│  Choose Cluster when:                                                │
│  • Data exceeds single node capacity                                │
│  • Need write scaling                                               │
│  • Can design around slot limitations                               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Production Considerations

### Network

```bash
# Cluster bus port = data port + 10000
# Node :7000 uses :17000 for cluster communication
# Ensure firewall allows both

# Bind to specific interface
bind 192.168.1.100
protected-mode no  # If binding to non-localhost

# Cluster configuration
cluster-announce-ip 192.168.1.100
cluster-announce-port 7000
cluster-announce-bus-port 17000
```

### Monitoring

```redis
# Sentinel monitoring
SENTINEL ckquorum mymaster
SENTINEL masters

# Cluster monitoring
CLUSTER INFO
CLUSTER NODES

# Key metrics to monitor:
# - cluster_state: ok/fail
# - connected_slaves
# - master_link_status
# - used_memory vs maxmemory
# - rejected_connections
```

### Backup Strategy

```bash
# RDB backup from one node per slot range
# Or use replicas for backup

# Cluster backup script
for port in 7000 7001 7002; do
  redis-cli -p $port --rdb /backup/dump-$port.rdb
done

# Restore requires recreating cluster with same slot allocation
```

---

## Summary

| Feature | Replication | Sentinel | Cluster |
|---------|-------------|----------|---------|
| Read Scaling | Yes | Yes | Yes |
| Write Scaling | No | No | Yes |
| Auto Failover | No | Yes | Yes |
| Data Partitioning | No | No | Yes |
| Multi-key Ops | Yes | Yes | Limited |

---

## Best Practices

```
Sentinel:
✓ Use at least 3 sentinels
✓ Deploy sentinels on separate machines
✓ Set reasonable down-after-milliseconds
✓ Configure notification scripts
✓ Use sentinel-aware clients

Cluster:
✓ Use at least 3 masters with 1 replica each
✓ Design with hash tags for related data
✓ Monitor cluster state continuously
✓ Plan resharding during low-traffic periods
✓ Keep slot distribution balanced
✓ Use cluster-aware client libraries
```
