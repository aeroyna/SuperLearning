# Consistency Models

## Introduction

Consistency models define the rules about when and how updates become visible to readers in a distributed system. Understanding these models is crucial for choosing the right guarantees for your application and understanding the behavior of distributed databases.

## Consistency Spectrum

```
┌─────────────────────────────────────────────────────────────┐
│                  Consistency Spectrum                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Strongest ◄────────────────────────────────► Weakest      │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                                                       │   │
│  │  Linearizable                                        │   │
│  │      │                                                │   │
│  │      ▼                                                │   │
│  │  Sequential Consistency                              │   │
│  │      │                                                │   │
│  │      ▼                                                │   │
│  │  Causal Consistency                                  │   │
│  │      │                                                │   │
│  │      ▼                                                │   │
│  │  Read-your-writes                                    │   │
│  │      │                                                │   │
│  │      ▼                                                │   │
│  │  Monotonic Reads                                     │   │
│  │      │                                                │   │
│  │      ▼                                                │   │
│  │  Eventual Consistency                                │   │
│  │                                                       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Stronger → Easier to reason about, higher latency         │
│  Weaker  → Harder to reason about, lower latency           │
└─────────────────────────────────────────────────────────────┘
```

## Linearizability (Strong Consistency)

```
┌─────────────────────────────────────────────────────────────┐
│                    Linearizability                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Definition: Operations appear to execute atomically at     │
│  some point between invocation and response                  │
│                                                              │
│  Also called: "Atomic consistency", "Strong consistency"    │
│                                                              │
│  Valid linearizable execution:                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  Client A: ────[write X=1]─────────────────────────│    │
│  │                    │                                 │    │
│  │                    ▼ (linearization point)          │    │
│  │                                                      │    │
│  │  Client B: ──────────────[read X]────────→ returns 1│    │
│  │                                                      │    │
│  │  The read started after write completed,            │    │
│  │  so it MUST see X=1                                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Invalid (violates linearizability):                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  Client A: ────[write X=1]────────────────────────  │    │
│  │                                                      │    │
│  │  Client B: ──────────────[read X]────→ returns 1   │    │
│  │                                                      │    │
│  │  Client C: ───────────────────[read X]─→ returns 0 │    │
│  │                                                      │    │
│  │  INVALID! C reads after B, but sees older value    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Examples: ZooKeeper, etcd, Spanner, CockroachDB           │
│  Cost: Requires coordination (consensus), higher latency   │
└─────────────────────────────────────────────────────────────┘
```

## Sequential Consistency

```
┌─────────────────────────────────────────────────────────────┐
│                 Sequential Consistency                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Definition: All operations appear to execute in some       │
│  sequential order that respects program order per client    │
│                                                              │
│  Key difference from linearizable:                          │
│  - Linearizable: Real-time ordering matters                 │
│  - Sequential: Only program order per client matters        │
│                                                              │
│  Valid sequential execution:                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Real time:                                          │    │
│  │  A: write(X,1)  read(Y)→0                           │    │
│  │  B:       write(Y,1)  read(X)→0                     │    │
│  │                                                      │    │
│  │  One valid sequential order:                        │    │
│  │  write(X,1) → write(Y,1) → read(Y)→1 → read(X)→1   │    │
│  │                                                      │    │
│  │  Another valid order:                                │    │
│  │  read(Y)→0 → write(X,1) → read(X)→0 → write(Y,1)   │    │
│  │                                                      │    │
│  │  Both preserve per-client order                     │    │
│  │  NOT linearizable (real-time violated)              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Weaker than linearizable but still strong                  │
│  Used in: Some memory consistency models                    │
└─────────────────────────────────────────────────────────────┘
```

## Causal Consistency

```
┌─────────────────────────────────────────────────────────────┐
│                   Causal Consistency                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Definition: Operations that are causally related are seen │
│  in the same order by all processes                          │
│                                                              │
│  Causality: A happened-before B if:                         │
│  1. A and B are on same process, A before B                 │
│  2. A is a send, B is corresponding receive                 │
│  3. Transitive: A→B, B→C implies A→C                        │
│                                                              │
│  Example (valid causal):                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  A: write(X,1)                                       │    │
│  │         │                                            │    │
│  │         └──(message with X=1)──→ B: read(X)=1       │    │
│  │                                      │               │    │
│  │                                      write(Y,2)      │    │
│  │                                                      │    │
│  │  X=1 causally precedes Y=2                          │    │
│  │  All processes see X=1 before Y=2                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Concurrent writes (no causal relation):                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  A: write(X,1)       (concurrent)    B: write(Y,2)  │    │
│  │                                                      │    │
│  │  C might see: X=1 then Y=2                          │    │
│  │  D might see: Y=2 then X=1                          │    │
│  │                                                      │    │
│  │  Both valid! No causal order between X and Y        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Implementation: Vector clocks, dependency tracking         │
│  Examples: MongoDB (causal sessions), COPS                  │
└─────────────────────────────────────────────────────────────┘
```

## Session Guarantees

```
┌─────────────────────────────────────────────────────────────┐
│                   Session Guarantees                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  READ YOUR WRITES:                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ A client always sees its own writes                 │    │
│  │                                                      │    │
│  │ Client: write(X,1) ──────→ read(X) = 1 (guaranteed)│    │
│  │                                                      │    │
│  │ Without this: "I updated my profile but it's still │    │
│  │ showing the old data!"                              │    │
│  │                                                      │    │
│  │ Implementation: Sticky sessions, version tracking   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  MONOTONIC READS:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Successive reads return same or newer values        │    │
│  │                                                      │    │
│  │ Client: read(X)=5 ────→ read(X) ≥ 5 (guaranteed)   │    │
│  │                                                      │    │
│  │ Without this: "Price was $100, refreshed, now $120, │    │
│  │ refreshed again, now $100?!"                        │    │
│  │                                                      │    │
│  │ Implementation: Version tracking, read from same   │    │
│  │ replica or wait for replica to catch up            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  MONOTONIC WRITES:                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Writes from same client applied in order            │    │
│  │                                                      │    │
│  │ Client: write(X,1) → write(X,2)                    │    │
│  │ All replicas see writes in this order               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  WRITES FOLLOW READS:                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Write is ordered after reads that influenced it    │    │
│  │                                                      │    │
│  │ Client: read(X)=1 → write(Y, f(X))                 │    │
│  │ Y write ordered after X was 1                       │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Eventual Consistency

```
┌─────────────────────────────────────────────────────────────┐
│                  Eventual Consistency                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Definition: If no new updates, all replicas eventually     │
│  converge to the same value                                  │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Time →                                              │    │
│  │                                                      │    │
│  │  Write X=5 at t=0                                   │    │
│  │                                                      │    │
│  │  Replica A: ──5───────────────────────────→        │    │
│  │  Replica B: ──0──0──5─────────────────────→        │    │
│  │  Replica C: ──0──0──0──0──5───────────────→        │    │
│  │                        ↑                            │    │
│  │                   Convergence point                 │    │
│  │                                                      │    │
│  │  • Before convergence: Different values possible   │    │
│  │  • After convergence: All see X=5                  │    │
│  │  • No guarantee on convergence time                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Flavors:                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ STRONG EVENTUAL CONSISTENCY:                        │    │
│  │ • Same set of updates → same final state           │    │
│  │ • No conflicts to resolve                           │    │
│  │ • Enabled by CRDTs                                  │    │
│  │                                                      │    │
│  │ EVENTUAL CONSISTENCY:                               │    │
│  │ • May need conflict resolution                      │    │
│  │ • LWW, vector clocks, application logic            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Examples: DNS, Cassandra, DynamoDB, S3                     │
└─────────────────────────────────────────────────────────────┘
```

## CRDTs (Conflict-free Replicated Data Types)

```
┌─────────────────────────────────────────────────────────────┐
│                        CRDTs                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Definition: Data structures that can be replicated and    │
│  updated concurrently, always converging to correct state   │
│                                                              │
│  G-COUNTER (Grow-only counter):                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Each node maintains its own count                  │    │
│  │  Node A: {A:5}                                       │    │
│  │  Node B: {A:3, B:7}                                  │    │
│  │  Node C: {A:5, B:7, C:2}                            │    │
│  │                                                      │    │
│  │  Merge: Take max of each key                        │    │
│  │  {A:5, B:7, C:2} → Total = 5+7+2 = 14              │    │
│  │                                                      │    │
│  │  Always converges, order doesn't matter             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  LWW-REGISTER (Last-Writer-Wins):                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Each update has timestamp                          │    │
│  │  Node A: (value=5, ts=100)                          │    │
│  │  Node B: (value=7, ts=105)                          │    │
│  │                                                      │    │
│  │  Merge: Take value with highest timestamp           │    │
│  │  Result: value=7 (ts=105 > ts=100)                 │    │
│  │                                                      │    │
│  │  ⚠ May lose concurrent updates                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  OR-SET (Observed-Remove Set):                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Each element has unique tag                        │    │
│  │  Add: Insert element with new tag                   │    │
│  │  Remove: Remove all observed tags for element      │    │
│  │                                                      │    │
│  │  Concurrent add + remove = element present          │    │
│  │  (add wins, because different tag)                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Used in: Riak, Redis CRDT, Automerge, Yjs                 │
└─────────────────────────────────────────────────────────────┘
```

## Consistency in Popular Databases

```
┌─────────────────────────────────────────────────────────────┐
│              Database Consistency Options                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CASSANDRA:                                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Tunable per query                                    │    │
│  │ ONE:        Eventual (fastest)                       │    │
│  │ QUORUM:     Strong (W+R > N)                        │    │
│  │ ALL:        Linearizable (but unavailable if 1 down)│    │
│  │ LOCAL_*:    Datacenter-aware variants               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  MONGODB:                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Write Concern:                                       │    │
│  │   w:1        Primary ack (default)                  │    │
│  │   w:majority Majority ack                           │    │
│  │   w:"all"    All replicas                           │    │
│  │                                                      │    │
│  │ Read Concern:                                        │    │
│  │   local      May read uncommitted                   │    │
│  │   majority   Read committed to majority             │    │
│  │   linearizable Strong consistency                   │    │
│  │                                                      │    │
│  │ Causal Sessions: Causal consistency                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  DYNAMODB:                                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Eventually consistent reads (default)              │    │
│  │ Strongly consistent reads (optional)               │    │
│  │ Transactions (ACID across items)                   │    │
│  │ Global Tables: Eventually consistent cross-region  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COCKROACHDB / SPANNER:                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Serializable by default                             │    │
│  │ External consistency (linearizable)                 │    │
│  │ Uses Raft/Paxos for consensus                      │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Choosing a Consistency Model

```
┌─────────────────────────────────────────────────────────────┐
│                 Consistency Selection Guide                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Use Linearizable/Strong when:                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Financial transactions                            │    │
│  │ • Inventory systems (prevent overselling)          │    │
│  │ • Unique constraints (usernames, IDs)              │    │
│  │ • Leader election, distributed locks               │    │
│  │ • Any "compare-and-swap" operation                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use Causal when:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Social media (see post before comments on it)   │    │
│  │ • Collaborative editing (see changes in order)    │    │
│  │ • Multi-step workflows                              │    │
│  │ • When eventual is too weak, strong too expensive  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use Eventual when:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Shopping carts (can merge later)                 │    │
│  │ • View counts, likes (approximate OK)              │    │
│  │ • DNS (TTL-based propagation)                      │    │
│  │ • Caching layers                                    │    │
│  │ • Analytics data                                    │    │
│  │ • When availability >> consistency                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  General rule: Use weakest model that meets requirements   │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **Consistency is a spectrum** - Not binary, many levels exist
2. **Stronger = slower** - Coordination required for guarantees
3. **Linearizable is strongest** - Real-time ordering, highest cost
4. **Causal preserves cause-effect** - Good middle ground
5. **Eventual is weakest but fastest** - May see stale or conflicting data
6. **CRDTs enable strong eventual** - Mathematically guaranteed convergence
