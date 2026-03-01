# PACELC Theorem

## Introduction

PACELC extends CAP by addressing what happens during normal operation (no partition). While CAP focuses on the partition scenario, PACELC recognizes that even without partitions, distributed systems must trade off between latency and consistency. Proposed by Daniel Abadi in 2010, it provides a more complete framework for understanding distributed system trade-offs.

## The PACELC Statement

```
┌─────────────────────────────────────────────────────────────┐
│                    PACELC Theorem                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  IF there is a Partition (P):                               │
│      Choose between Availability (A) and Consistency (C)    │
│  ELSE (E) - normal operation:                               │
│      Choose between Latency (L) and Consistency (C)         │
│                                                              │
│  Written as: P → A/C, E → L/C                               │
│                                                              │
│  Examples:                                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ PA/EL: Partition → Available, Else → Low Latency   │    │
│  │        (Dynamo, Cassandra with CL=ONE)              │    │
│  │                                                      │    │
│  │ PC/EC: Partition → Consistent, Else → Consistent   │    │
│  │        (HBase, traditional RDBMS)                   │    │
│  │                                                      │    │
│  │ PA/EC: Partition → Available, Else → Consistent    │    │
│  │        (MongoDB default, unlikely combo)            │    │
│  │                                                      │    │
│  │ PC/EL: Partition → Consistent, Else → Low Latency  │    │
│  │        (Yahoo PNUTS - Yahoo's old system)          │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Why CAP Isn't Enough

```
┌─────────────────────────────────────────────────────────────┐
│                 CAP's Limitation                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CAP only describes partition scenario:                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ During Partition:                                    │    │
│  │   CP → Consistent, may be unavailable               │    │
│  │   AP → Available, may be inconsistent               │    │
│  │                                                      │    │
│  │ Normal Operation (99%+ of time):                    │    │
│  │   CAP says nothing!                                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  But normal operation has trade-offs too:                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  Strong Consistency:                                 │    │
│  │  ┌────────────────────────────────────────────────┐ │    │
│  │  │ Client                                         │ │    │
│  │  │   │                                            │ │    │
│  │  │   ├── Write X=5 ──→ Leader                    │ │    │
│  │  │   │                   │                        │ │    │
│  │  │   │              Replicate to followers        │ │    │
│  │  │   │                   │    │    │              │ │    │
│  │  │   │                   ▼    ▼    ▼              │ │    │
│  │  │   │               Wait for majority ack       │ │    │
│  │  │   │                   │                        │ │    │
│  │  │   ◄─────── OK ────────┘                       │ │    │
│  │  │                                                │ │    │
│  │  │   Latency: ~10-100ms (network round trips)   │ │    │
│  │  └────────────────────────────────────────────────┘ │    │
│  │                                                      │    │
│  │  Eventual Consistency:                               │    │
│  │  ┌────────────────────────────────────────────────┐ │    │
│  │  │ Client                                         │ │    │
│  │  │   │                                            │ │    │
│  │  │   ├── Write X=5 ──→ Local node                │ │    │
│  │  │   │                   │                        │ │    │
│  │  │   ◄─────── OK ────────┘ (immediate)           │ │    │
│  │  │                   │                            │ │    │
│  │  │              Async replicate                   │ │    │
│  │  │                   ▼                            │ │    │
│  │  │               Other nodes (later)             │ │    │
│  │  │                                                │ │    │
│  │  │   Latency: ~1-5ms (local only)               │ │    │
│  │  └────────────────────────────────────────────────┘ │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## PACELC Classifications

### PA/EL Systems

```
┌─────────────────────────────────────────────────────────────┐
│                  PA/EL Systems                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  During Partition: Prioritize Availability                  │
│  Normal Operation: Prioritize Low Latency                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ DYNAMO / DYNAMODB:                                  │    │
│  │ • Write to any node, return immediately            │    │
│  │ • Async replication to other nodes                 │    │
│  │ • Sloppy quorum during partitions                  │    │
│  │ • Vector clocks for conflict detection             │    │
│  │                                                      │    │
│  │ CASSANDRA (CL=ONE):                                 │    │
│  │ • Single node ack for writes                        │    │
│  │ • Hinted handoff during partitions                 │    │
│  │ • Eventually consistent                             │    │
│  │ • Sub-millisecond latency                           │    │
│  │                                                      │    │
│  │ COUCHDB:                                            │    │
│  │ • Multi-master replication                          │    │
│  │ • Accept writes during partition                   │    │
│  │ • Conflict resolution on read                       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use Cases:                                                  │
│  • Shopping carts                                            │
│  • User session data                                         │
│  • Social media feeds                                        │
│  • Gaming leaderboards                                       │
└─────────────────────────────────────────────────────────────┘
```

### PC/EC Systems

```
┌─────────────────────────────────────────────────────────────┐
│                  PC/EC Systems                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  During Partition: Prioritize Consistency (unavailable)    │
│  Normal Operation: Prioritize Consistency (higher latency) │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ HBASE:                                              │    │
│  │ • Single master for each region                     │    │
│  │ • No writes if master unreachable                   │    │
│  │ • Synchronous replication                           │    │
│  │ • Strong consistency always                         │    │
│  │                                                      │    │
│  │ ZOOKEEPER:                                          │    │
│  │ • Majority quorum required                          │    │
│  │ • Minority partition read-only                      │    │
│  │ • Sync replication before ack                       │    │
│  │                                                      │    │
│  │ TRADITIONAL RDBMS (distributed):                    │    │
│  │ • Two-phase commit                                  │    │
│  │ • Distributed locks                                 │    │
│  │ • High latency for consistency                      │    │
│  │                                                      │    │
│  │ SPANNER:                                            │    │
│  │ • TrueTime for global ordering                      │    │
│  │ • External consistency (linearizable)               │    │
│  │ • Higher latency for global transactions           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use Cases:                                                  │
│  • Financial transactions                                    │
│  • Inventory systems                                         │
│  • Distributed locks                                         │
│  • Coordination services                                     │
└─────────────────────────────────────────────────────────────┘
```

### PA/EC Systems

```
┌─────────────────────────────────────────────────────────────┐
│                  PA/EC Systems                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  During Partition: Prioritize Availability                  │
│  Normal Operation: Prioritize Consistency                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ MONGODB (default settings):                         │    │
│  │ • w=1 (primary ack) - fast                         │    │
│  │ • Reads from primary - consistent                  │    │
│  │ • During partition: Accept writes on old primary   │    │
│  │ • Rollback when partition heals                    │    │
│  │                                                      │    │
│  │ Note: With w=majority, becomes more PC/EC           │    │
│  │                                                      │    │
│  │ POSTGRESQL with async replication:                  │    │
│  │ • Primary accepts all writes                        │    │
│  │ • Async streaming to replicas                      │    │
│  │ • Failover may lose some writes                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Less common because:                                        │
│  • Consistency during normal op usually means              │
│    you want it during partitions too                        │
│  • May indicate inconsistent design                         │
└─────────────────────────────────────────────────────────────┘
```

### PC/EL Systems

```
┌─────────────────────────────────────────────────────────────┐
│                  PC/EL Systems                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  During Partition: Prioritize Consistency                   │
│  Normal Operation: Prioritize Low Latency                   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ YAHOO PNUTS (historical):                           │    │
│  │ • Per-record consistency                            │    │
│  │ • Async replication normally (low latency)         │    │
│  │ • Disable writes during partition                   │    │
│  │ • "Timeline consistency" model                     │    │
│  │                                                      │    │
│  │ MEGASTORE (Google):                                 │    │
│  │ • Entity groups for consistency                     │    │
│  │ • Optimistic concurrency                            │    │
│  │ • Async cross-group operations                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use Cases:                                                  │
│  • Read-heavy with occasional critical writes              │
│  • Per-user data (user is partition)                       │
│  • When partition is rare but serious                       │
└─────────────────────────────────────────────────────────────┘
```

## Latency vs Consistency Trade-off

```
┌─────────────────────────────────────────────────────────────┐
│              Latency-Consistency Spectrum                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Low Latency ◄──────────────────────────► High Consistency │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                                                        │  │
│  │  CL=ONE        CL=QUORUM       CL=ALL                 │  │
│  │  ~1ms          ~5-20ms         ~50-100ms              │  │
│  │    │              │               │                    │  │
│  │    ▼              ▼               ▼                    │  │
│  │  ┌────┐       ┌─────────┐     ┌───────────┐          │  │
│  │  │Ack │       │ Wait for│     │Wait for   │          │  │
│  │  │from│       │ majority│     │all nodes  │          │  │
│  │  │1   │       │ of nodes│     │           │          │  │
│  │  └────┘       └─────────┘     └───────────┘          │  │
│  │                                                        │  │
│  │  Risk:         Balanced        Risk:                  │  │
│  │  Data loss     compromise      Unavailability         │  │
│  │  on failure                    if any node down       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  Cassandra example (3 replicas):                            │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ Write CL │ Read CL  │ Consistent? │ Latency          │  │
│  │──────────│──────────│─────────────│──────────────────│  │
│  │ ONE      │ ONE      │ No          │ Very low         │  │
│  │ ONE      │ ALL      │ No          │ Medium           │  │
│  │ QUORUM   │ ONE      │ No          │ Medium           │  │
│  │ QUORUM   │ QUORUM   │ Yes         │ Medium-High      │  │
│  │ ALL      │ ONE      │ Yes         │ High             │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  Formula: W + R > N for consistency                         │
│  (Write nodes + Read nodes > Total replicas)               │
└─────────────────────────────────────────────────────────────┘
```

## Practical Implications

```
┌─────────────────────────────────────────────────────────────┐
│                 Design Considerations                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. KNOW YOUR REQUIREMENTS:                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Question                     │ Answer → Choice      │    │
│  │─────────────────────────────│──────────────────────│    │
│  │ Can you lose writes?        │ No → PC              │    │
│  │ Can users see stale data?   │ No → EC              │    │
│  │ Need sub-10ms latency?      │ Yes → EL             │    │
│  │ Must always be writable?    │ Yes → PA             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  2. DIFFERENT DATA, DIFFERENT CHOICES:                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Data Type              │ Suggested Classification  │    │
│  │────────────────────────│──────────────────────────│    │
│  │ User account balance   │ PC/EC                     │    │
│  │ Shopping cart          │ PA/EL                     │    │
│  │ View counts            │ PA/EL                     │    │
│  │ Order placement        │ PC/EC                     │    │
│  │ Product catalog        │ PA/EC                     │    │
│  │ Session data           │ PA/EL                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  3. OPERATIONAL REALITY:                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Monitor partition frequency                       │    │
│  │ • Measure actual latencies                          │    │
│  │ • Track consistency violations                      │    │
│  │ • Have conflict resolution strategy                 │    │
│  │ • Test partition behavior                           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## System Classification Summary

```
┌─────────────────────────────────────────────────────────────┐
│              PACELC System Classifications                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  System          │ PACELC   │ Notes                         │
│  ────────────────│──────────│───────────────────────────────│
│  Cassandra       │ PA/EL*   │ *Tunable with CL              │
│  DynamoDB        │ PA/EL    │ Default eventual consistency  │
│  Riak            │ PA/EL    │ Sloppy quorum, LWW           │
│  CouchDB         │ PA/EL    │ Multi-master, MVCC           │
│  ────────────────│──────────│───────────────────────────────│
│  HBase           │ PC/EC    │ Single region master          │
│  ZooKeeper       │ PC/EC    │ Majority quorum               │
│  etcd            │ PC/EC    │ Raft consensus                │
│  Spanner         │ PC/EC    │ TrueTime, 2PC                 │
│  CockroachDB     │ PC/EC    │ Serializable, Raft           │
│  ────────────────│──────────│───────────────────────────────│
│  MongoDB         │ PA/EC*   │ *Depends on write concern     │
│  PostgreSQL      │ PA/EC*   │ *Depends on replication mode  │
│  ────────────────│──────────│───────────────────────────────│
│  PNUTS           │ PC/EL    │ Yahoo (historical)            │
│  ────────────────│──────────│───────────────────────────────│
│                                                              │
│  * = Tunable based on configuration                         │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **CAP is incomplete** - Doesn't address normal operation trade-offs
2. **Latency matters** - Users care about response time always
3. **Four classifications** - PA/EL, PA/EC, PC/EL, PC/EC
4. **Many systems are tunable** - Configuration changes classification
5. **Match to requirements** - Different data may need different choices
6. **Normal operation is common** - EL vs EC affects everyday performance
