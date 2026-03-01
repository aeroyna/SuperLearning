# NewSQL and Distributed SQL

## Introduction

NewSQL databases combine the ACID guarantees of traditional relational databases with the horizontal scalability of NoSQL systems. They aim to provide the best of both worlds: SQL compatibility and strong consistency with distributed architecture.

## Topics in This Section

1. **[CockroachDB](01_cockroachdb.md)**
2. **[TiDB](02_tidb.md)**
3. **[Spanner and F1](03_spanner_and_f1.md)**
4. **[Vitess](04_vitess.md)**

## NewSQL vs Traditional vs NoSQL

```
┌─────────────────────────────────────────────────────────────┐
│              Database Category Comparison                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TRADITIONAL RDBMS (MySQL, PostgreSQL):                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✓ ACID transactions                                 │    │
│  │ ✓ SQL support                                       │    │
│  │ ✓ Strong consistency                                │    │
│  │ ✗ Limited horizontal scaling                        │    │
│  │ ✗ Single-node bottleneck                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  NOSQL (MongoDB, Cassandra):                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✓ Horizontal scaling                                │    │
│  │ ✓ High availability                                 │    │
│  │ ✓ Flexible schema                                   │    │
│  │ ✗ Limited transaction support                       │    │
│  │ ✗ Eventual consistency (usually)                    │    │
│  │ ✗ No standard query language                        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  NEWSQL (CockroachDB, TiDB, Spanner):                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✓ ACID transactions                                 │    │
│  │ ✓ SQL support                                       │    │
│  │ ✓ Horizontal scaling                                │    │
│  │ ✓ Strong consistency                                │    │
│  │ ✓ High availability                                 │    │
│  │ ~ More complex architecture                         │    │
│  │ ~ Higher latency than single-node                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Key NewSQL Architectures

```
┌─────────────────────────────────────────────────────────────┐
│                NewSQL Architecture Patterns                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  SHARED-NOTHING (Most NewSQL):                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │         ┌─────────────────────────────────┐         │    │
│  │         │        Query Router              │         │    │
│  │         └─────────────┬───────────────────┘         │    │
│  │                       │                             │    │
│  │      ┌────────────────┼────────────────┐           │    │
│  │      │                │                │            │    │
│  │  ┌───▼───┐        ┌───▼───┐        ┌───▼───┐       │    │
│  │  │Node 1 │        │Node 2 │        │Node 3 │       │    │
│  │  │Data A │        │Data B │        │Data C │       │    │
│  │  │Compute│        │Compute│        │Compute│       │    │
│  │  └───────┘        └───────┘        └───────┘       │    │
│  │                                                     │    │
│  │  Each node owns data + compute                     │    │
│  │  Used by: CockroachDB, TiDB (TiKV layer)          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COMPUTE-STORAGE SEPARATION:                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  ┌────────────────────────────────────────────┐     │    │
│  │  │           Compute Layer                     │     │    │
│  │  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐   │     │    │
│  │  │  │SQL 1 │  │SQL 2 │  │SQL 3 │  │SQL N │   │     │    │
│  │  │  └──────┘  └──────┘  └──────┘  └──────┘   │     │    │
│  │  └────────────────────┬───────────────────────┘     │    │
│  │                       │                             │    │
│  │  ┌────────────────────▼───────────────────────┐     │    │
│  │  │           Storage Layer                     │     │    │
│  │  │    Distributed, Replicated Storage         │     │    │
│  │  └────────────────────────────────────────────┘     │    │
│  │                                                     │    │
│  │  Scale compute and storage independently           │    │
│  │  Used by: TiDB (TiDB+TiKV), Spanner               │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Distributed Transaction Techniques

```
┌─────────────────────────────────────────────────────────────┐
│           Distributed Transaction Methods                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TWO-PHASE COMMIT (2PC):                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Coordinator                                          │    │
│  │     │                                                │    │
│  │     ├──→ PREPARE ──→ Participant 1 ──→ VOTE YES    │    │
│  │     ├──→ PREPARE ──→ Participant 2 ──→ VOTE YES    │    │
│  │     │                                                │    │
│  │     │   All voted YES?                              │    │
│  │     │                                                │    │
│  │     ├──→ COMMIT  ──→ Participant 1 ──→ ACK         │    │
│  │     └──→ COMMIT  ──→ Participant 2 ──→ ACK         │    │
│  │                                                      │    │
│  │  ✓ Guarantees atomicity                             │    │
│  │  ✗ Blocking if coordinator fails                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  PERCOLATOR (TiDB):                                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Optimistic concurrency with timestamp ordering      │    │
│  │ 1. Prewrite: Write tentative values with locks     │    │
│  │ 2. Commit: Clear locks, make values visible        │    │
│  │ Conflict detection at commit time                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TRUETIME (Spanner):                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ GPS + atomic clocks provide bounded time           │    │
│  │ Timestamp ordering without coordination            │    │
│  │ Wait out uncertainty interval before commit        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## When to Choose NewSQL

```
┌─────────────────────────────────────────────────────────────┐
│               NewSQL Decision Guide                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CHOOSE NEWSQL WHEN:                                        │
│  • Need ACID + horizontal scaling                          │
│  • Outgrowing single-node database                         │
│  • Want SQL compatibility for existing apps                │
│  • Require strong consistency across regions               │
│  • Need automatic failover and rebalancing                 │
│                                                              │
│  CHOOSE TRADITIONAL RDBMS WHEN:                             │
│  • Single-node performance is sufficient                   │
│  • Simpler operations preferred                            │
│  • Lower latency required                                  │
│  • Existing expertise and tooling                          │
│                                                              │
│  CHOOSE NOSQL WHEN:                                         │
│  • Flexible schema required                                │
│  • Eventual consistency acceptable                         │
│  • Specific data model (graph, document, time-series)      │
│  • Ultra-low latency needed                                │
└─────────────────────────────────────────────────────────────┘
```
