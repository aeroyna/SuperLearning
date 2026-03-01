# Distributed Systems Fundamentals

## Introduction

Distributed databases span multiple machines to achieve scalability, availability, and fault tolerance that single-node systems cannot provide. Understanding the fundamental trade-offs and theoretical foundations is essential for designing and operating distributed data systems.

## Why Distributed Databases?

```
┌─────────────────────────────────────────────────────────────┐
│              Drivers for Distribution                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  SCALABILITY                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Vertical scaling has limits (biggest server)      │    │
│  │ • Horizontal scaling: add more commodity hardware   │    │
│  │ • Read scaling: replicas serve read traffic        │    │
│  │ • Write scaling: partition data across nodes       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  AVAILABILITY                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Single node = single point of failure             │    │
│  │ • Replication enables failover                      │    │
│  │ • Geographic distribution for disaster recovery    │    │
│  │ • 99.99% uptime = 52 min downtime/year             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  LATENCY                                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Speed of light: ~100ms round-trip US ↔ Europe    │    │
│  │ • Place data closer to users                        │    │
│  │ • CDN for static content, database replicas for data│    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Fundamental Challenges

```
┌─────────────────────────────────────────────────────────────┐
│              The Eight Fallacies of Distributed Computing    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. The network is reliable                                  │
│     Reality: Packets drop, connections timeout              │
│                                                              │
│  2. Latency is zero                                          │
│     Reality: Cross-datacenter: 10-100ms, Cross-region: 100ms+│
│                                                              │
│  3. Bandwidth is infinite                                    │
│     Reality: Network saturation is real                     │
│                                                              │
│  4. The network is secure                                    │
│     Reality: Encryption and auth are essential              │
│                                                              │
│  5. Topology doesn't change                                  │
│     Reality: Nodes join/leave, routes change                │
│                                                              │
│  6. There is one administrator                               │
│     Reality: Multi-team, multi-org complexity               │
│                                                              │
│  7. Transport cost is zero                                   │
│     Reality: Serialization, bandwidth have costs            │
│                                                              │
│  8. The network is homogeneous                               │
│     Reality: Mixed hardware, protocols, versions            │
└─────────────────────────────────────────────────────────────┘
```

## Core Concepts

### Data Distribution Strategies

```
┌─────────────────────────────────────────────────────────────┐
│                  Data Distribution                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  REPLICATION: Same data on multiple nodes                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │    Node A          Node B          Node C           │    │
│  │  ┌───────┐       ┌───────┐       ┌───────┐         │    │
│  │  │ Data  │       │ Data  │       │ Data  │         │    │
│  │  │ Copy  │       │ Copy  │       │ Copy  │         │    │
│  │  └───────┘       └───────┘       └───────┘         │    │
│  │                                                      │    │
│  │  Purpose: Fault tolerance, read scaling             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  PARTITIONING (Sharding): Different data on different nodes │
│  ┌─────────────────────────────────────────────────────┐    │
│  │    Node A          Node B          Node C           │    │
│  │  ┌───────┐       ┌───────┐       ┌───────┐         │    │
│  │  │Users  │       │Users  │       │Users  │         │    │
│  │  │ A-H   │       │ I-P   │       │ Q-Z   │         │    │
│  │  └───────┘       └───────┘       └───────┘         │    │
│  │                                                      │    │
│  │  Purpose: Storage scaling, write scaling            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COMBINATION: Partitioned + Replicated                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Partition 1     Partition 2     Partition 3        │    │
│  │  ┌─────────┐     ┌─────────┐     ┌─────────┐       │    │
│  │  │ Node A  │     │ Node B  │     │ Node C  │ Primary│    │
│  │  │ Node B  │     │ Node C  │     │ Node A  │ Replica│    │
│  │  │ Node C  │     │ Node A  │     │ Node B  │ Replica│    │
│  │  └─────────┘     └─────────┘     └─────────┘       │    │
│  │                                                      │    │
│  │  Each partition on 3 nodes for fault tolerance      │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Topics Covered

1. **[CAP Theorem](01_cap_theorem.md)** - Consistency, Availability, Partition tolerance trade-offs
2. **[PACELC Theorem](02_pacelc_theorem.md)** - Extending CAP with latency considerations
3. **[Consistency Models](03_consistency_models.md)** - From linearizability to eventual consistency
4. **[Consensus Algorithms](04_consensus_algorithms.md)** - Paxos, Raft, and Byzantine fault tolerance

## Network Partitions

```
┌─────────────────────────────────────────────────────────────┐
│                   Network Partitions                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal Operation:                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │    Node A ←──────→ Node B ←──────→ Node C           │    │
│  │              ↖         ↕         ↗                   │    │
│  │               ←───── Node D ────→                    │    │
│  │                                                      │    │
│  │  All nodes can communicate                           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Network Partition:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │    Node A ←──────→ Node B    ╳    Node C            │    │
│  │              ↖         ↕      ╳      ↕               │    │
│  │               ←───── Node D ─╳──────→               │    │
│  │                                                      │    │
│  │  Partition: {A, B, D} cannot reach {C}              │    │
│  │                                                      │    │
│  │  What happens to writes?                             │    │
│  │  • CP system: Refuse writes on minority side        │    │
│  │  • AP system: Accept writes, reconcile later        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Real-world causes:                                          │
│  • Switch/router failure                                     │
│  • Network cable cut                                         │
│  • Datacenter network issues                                 │
│  • Cloud provider AZ isolation                               │
│  • Asymmetric partitions (A→B works, B→A doesn't)          │
└─────────────────────────────────────────────────────────────┘
```

## Time in Distributed Systems

```
┌─────────────────────────────────────────────────────────────┐
│              Time and Ordering Challenges                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  PHYSICAL CLOCKS:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Problem: Clocks drift, NTP isn't perfect            │    │
│  │                                                      │    │
│  │ Node A: 12:00:00.000                                 │    │
│  │ Node B: 12:00:00.050 (50ms ahead)                   │    │
│  │ Node C: 11:59:59.980 (20ms behind)                  │    │
│  │                                                      │    │
│  │ Event at A (12:00:00.010) vs B (12:00:00.040)       │    │
│  │ Which happened first? Clock skew makes it unclear! │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  LOGICAL CLOCKS (Lamport):                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Rule: If A happens-before B, then LC(A) < LC(B)     │    │
│  │                                                      │    │
│  │ A────[1]────[2]─────────[4]────→                    │    │
│  │              │           ↑                           │    │
│  │              ↓           │                           │    │
│  │ B──────────[3]──────────[5]────→                    │    │
│  │                                                      │    │
│  │ Send: increment and send clock                       │    │
│  │ Receive: max(local, received) + 1                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  VECTOR CLOCKS:                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Captures causality between all nodes                │    │
│  │                                                      │    │
│  │ A────[1,0,0]──[2,0,0]─────────[2,2,0]────→          │    │
│  │                 │               ↑                    │    │
│  │                 ↓               │                    │    │
│  │ B─────────────[2,1,0]────────[2,2,0]────→           │    │
│  │                                                      │    │
│  │ Concurrent events: Neither < other                  │    │
│  │ [1,2,0] vs [2,1,0] - concurrent!                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Failure Modes

```
┌─────────────────────────────────────────────────────────────┐
│                     Failure Types                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CRASH FAILURES:                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Node stops responding (hardware failure, OOM, etc)  │    │
│  │ • Fail-stop: Node crashes and stays down           │    │
│  │ • Fail-recover: Node crashes, restarts later       │    │
│  │ Solution: Replication, health checks, failover     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  OMISSION FAILURES:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Node fails to send/receive messages                 │    │
│  │ • Send omission: Message not sent                   │    │
│  │ • Receive omission: Message not received           │    │
│  │ Solution: Timeouts, retries, acknowledgments       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TIMING FAILURES:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Node responds too slowly                            │    │
│  │ • GC pause, CPU saturation, slow disk              │    │
│  │ • Indistinguishable from crash (until response)    │    │
│  │ Solution: Timeouts, circuit breakers               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  BYZANTINE FAILURES:                                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Node behaves maliciously or arbitrarily            │    │
│  │ • Sends conflicting messages to different nodes    │    │
│  │ • Corrupts data, lies about state                  │    │
│  │ Solution: Byzantine fault tolerance (BFT) protocols│    │
│  │ Requires: 3f + 1 nodes to tolerate f Byzantine    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Distributed System Architectures

```
┌─────────────────────────────────────────────────────────────┐
│                 Common Architectures                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  PRIMARY-SECONDARY (Leader-Follower):                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │         ┌─────────┐                                 │    │
│  │  Writes │ Primary │ Reads+Writes                    │    │
│  │    ───→ │ (Leader)│ ←───                            │    │
│  │         └────┬────┘                                 │    │
│  │         ┌────┴────┐                                 │    │
│  │    ┌────▼───┐ ┌───▼────┐                           │    │
│  │    │Replica │ │Replica │  Reads only               │    │
│  │    └────────┘ └────────┘                           │    │
│  │                                                      │    │
│  │  Examples: PostgreSQL, MySQL, MongoDB               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  MULTI-PRIMARY (Multi-Leader):                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐         │    │
│  │  │Primary 1│←──→│Primary 2│←──→│Primary 3│         │    │
│  │  │(writes) │    │(writes) │    │(writes) │         │    │
│  │  └─────────┘    └─────────┘    └─────────┘         │    │
│  │                                                      │    │
│  │  Conflict resolution needed                          │    │
│  │  Examples: CouchDB, multi-datacenter MySQL          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  LEADERLESS:                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐         │    │
│  │  │  Node   │    │  Node   │    │  Node   │         │    │
│  │  │(writes) │    │(writes) │    │(writes) │         │    │
│  │  └─────────┘    └─────────┘    └─────────┘         │    │
│  │                                                      │    │
│  │  Quorum-based reads/writes                          │    │
│  │  Examples: Cassandra, DynamoDB, Riak                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **Distribution is a trade-off** - Gains in scale/availability have costs
2. **Network failures are inevitable** - Design for partition tolerance
3. **Time is unreliable** - Use logical clocks for ordering when needed
4. **Replication and partitioning are orthogonal** - Often used together
5. **Failure modes vary** - Crash vs Byzantine require different solutions
6. **No one-size-fits-all** - Choose architecture based on requirements
