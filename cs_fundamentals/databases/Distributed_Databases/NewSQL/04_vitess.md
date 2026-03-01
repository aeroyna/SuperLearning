# Vitess

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Vitess Overview                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Vitess: MySQL scaling middleware                           │
│  Originally developed at YouTube, now CNCF graduated        │
│                                                              │
│  Key idea: Shard MySQL horizontally while maintaining      │
│            MySQL compatibility                              │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Application                                          │    │
│  │     │                                                │    │
│  │     │ MySQL protocol                                │    │
│  │     ▼                                                │    │
│  │ ┌─────────────────────────────────────────────┐     │    │
│  │ │              VTGate                          │     │    │
│  │ │     Query routing and aggregation           │     │    │
│  │ └─────────────────────────────────────────────┘     │    │
│  │     │                                                │    │
│  │     ├──────────────┬───────────────┐                │    │
│  │     ▼              ▼               ▼                │    │
│  │ ┌────────┐    ┌────────┐    ┌────────┐             │    │
│  │ │VTTablet│    │VTTablet│    │VTTablet│             │    │
│  │ │Shard 0 │    │Shard 1 │    │Shard 2 │             │    │
│  │ └────────┘    └────────┘    └────────┘             │    │
│  │     │              │               │                │    │
│  │     ▼              ▼               ▼                │    │
│  │ ┌────────┐    ┌────────┐    ┌────────┐             │    │
│  │ │ MySQL  │    │ MySQL  │    │ MySQL  │             │    │
│  │ └────────┘    └────────┘    └────────┘             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Vitess adds horizontal scaling to MySQL                   │
└─────────────────────────────────────────────────────────────┘
```

## Architecture Components

```
┌─────────────────────────────────────────────────────────────┐
│                 Vitess Components                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  VTGATE:                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Stateless query router                            │    │
│  │ • MySQL protocol endpoint                           │    │
│  │ • Routes queries to appropriate shards              │    │
│  │ • Aggregates results from multiple shards          │    │
│  │ • Connection pooling                                │    │
│  │ • Query parsing and rewriting                       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  VTTABLET:                                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Runs alongside each MySQL instance                │    │
│  │ • Query rewriting and sanitization                  │    │
│  │ • Connection pooling to MySQL                       │    │
│  │ • Query caching                                     │    │
│  │ • Replication management                            │    │
│  │ • Online schema changes                             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  VTCTLD:                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Cluster management daemon                         │    │
│  │ • Topology management                               │    │
│  │ • Orchestrates resharding                          │    │
│  │ • Schema changes                                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TOPOLOGY SERVICE:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Stores cluster metadata                           │    │
│  │ • Uses etcd, ZooKeeper, or Consul                  │    │
│  │ • Keyspace, shard, tablet information              │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Sharding in Vitess

```
┌─────────────────────────────────────────────────────────────┐
│                 Vitess Sharding                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  KEYSPACE: Logical database (collection of shards)         │
│                                                              │
│  Sharding by Vindexes (Vitess Indexes):                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  Vindex Types:                                       │    │
│  │  • Hash: Distribute evenly (default)                │    │
│  │  • Range: Sequential distribution                   │    │
│  │  • Lookup: Custom mapping table                     │    │
│  │                                                      │    │
│  │  Example - Hash Sharding:                           │    │
│  │  ┌──────────────────────────────────────────────┐   │    │
│  │  │ CREATE TABLE users (                          │   │    │
│  │  │   id BIGINT PRIMARY KEY,                     │   │    │
│  │  │   name VARCHAR(100)                          │   │    │
│  │  │ );                                            │   │    │
│  │  │                                               │   │    │
│  │  │ Vindex: xxhash(id)                           │   │    │
│  │  │                                               │   │    │
│  │  │ Shard -80: hash < 0x80 (first half)         │   │    │
│  │  │ Shard 80-: hash >= 0x80 (second half)       │   │    │
│  │  └──────────────────────────────────────────────┘   │    │
│  │                                                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  VSCHEMA: Schema that defines sharding rules               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ {                                                    │    │
│  │   "tables": {                                       │    │
│  │     "users": {                                      │    │
│  │       "column_vindexes": [{                        │    │
│  │         "column": "id",                            │    │
│  │         "name": "xxhash"                           │    │
│  │       }]                                            │    │
│  │     }                                               │    │
│  │   }                                                 │    │
│  │ }                                                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Query Routing

```
┌─────────────────────────────────────────────────────────────┐
│                Query Routing Flow                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Single-Shard Query:                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ SELECT * FROM users WHERE id = 123;                 │    │
│  │                                                      │    │
│  │ 1. VTGate receives query                            │    │
│  │ 2. Compute vindex: xxhash(123) → 0x4A...           │    │
│  │ 3. Route to shard -80 (matches range)              │    │
│  │ 4. Return result directly                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Scatter-Gather Query:                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ SELECT COUNT(*) FROM users WHERE created > ?;       │    │
│  │                                                      │    │
│  │ 1. VTGate receives query                            │    │
│  │ 2. No shard key in WHERE clause                    │    │
│  │ 3. Send to ALL shards                               │    │
│  │ 4. Aggregate COUNT from each shard                  │    │
│  │ 5. Return combined result                          │    │
│  │                                                      │    │
│  │ VTGate                                               │    │
│  │   │                                                  │    │
│  │   ├──→ Shard -80:  COUNT = 50000                   │    │
│  │   └──→ Shard 80-:  COUNT = 48000                   │    │
│  │        ────────────────────────                     │    │
│  │        Result:     COUNT = 98000                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Cross-Shard Join:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ SELECT u.*, o.* FROM users u                        │    │
│  │   JOIN orders o ON u.id = o.user_id;               │    │
│  │                                                      │    │
│  │ If same sharding key: Local join on each shard     │    │
│  │ If different: Scatter-gather with aggregation      │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Resharding

```
┌─────────────────────────────────────────────────────────────┐
│                    Resharding                                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Online resharding without downtime:                        │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Before: 2 shards                                     │    │
│  │                                                      │    │
│  │  ┌───────────┐      ┌───────────┐                   │    │
│  │  │ Shard -80 │      │ Shard 80- │                   │    │
│  │  │  (50%)    │      │  (50%)    │                   │    │
│  │  └───────────┘      └───────────┘                   │    │
│  │                                                      │    │
│  │ After: 4 shards                                      │    │
│  │                                                      │    │
│  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐            │    │
│  │  │ -40  │  │40-80 │  │80-c0 │  │ c0-  │            │    │
│  │  │(25%) │  │(25%) │  │(25%) │  │(25%) │            │    │
│  │  └──────┘  └──────┘  └──────┘  └──────┘            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Resharding Steps:                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 1. Create new shards (empty)                        │    │
│  │ 2. VDiff: Copy initial data                         │    │
│  │ 3. VReplication: Stream ongoing changes            │    │
│  │ 4. Verify: Check data consistency                   │    │
│  │ 5. Switch reads: Route reads to new shards         │    │
│  │ 6. Switch writes: Route writes to new shards       │    │
│  │ 7. Cleanup: Remove old shards                       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Traffic switching is instant (no downtime)                │
└─────────────────────────────────────────────────────────────┘
```

## Key Features Summary

```
┌─────────────────────────────────────────────────────────────┐
│                 Vitess Key Features                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ✓ Horizontal scaling for MySQL                            │
│  ✓ MySQL wire protocol compatible                          │
│  ✓ Online resharding                                        │
│  ✓ Connection pooling and query routing                    │
│  ✓ Built-in replication management                         │
│  ✓ Online schema migrations                                │
│  ✓ Kubernetes native (Vitess Operator)                     │
│  ✓ CNCF graduated project                                  │
│                                                              │
│  Limitations:                                               │
│  • Cross-shard transactions are expensive                  │
│  • Some MySQL features unsupported                         │
│  • Requires understanding of sharding                      │
│  • Operational complexity                                   │
│                                                              │
│  Use Cases:                                                 │
│  • Scaling existing MySQL applications                     │
│  • Multi-tenant SaaS applications                          │
│  • High-traffic web applications                           │
│  • YouTube, Slack, Pinterest, Square use Vitess           │
└─────────────────────────────────────────────────────────────┘
```

## Vitess vs Other Solutions

```
┌─────────────────────────────────────────────────────────────┐
│              Comparison with Alternatives                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  VITESS vs MYSQL CLUSTER:                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Vitess: Proxy-based, uses standard MySQL           │    │
│  │ Cluster: Requires NDB storage engine               │    │
│  │ Vitess: More flexible deployment                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  VITESS vs TIDB:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Vitess: Adds sharding to MySQL                      │    │
│  │ TiDB: Complete NewSQL database                      │    │
│  │ Vitess: Manual shard key design                     │    │
│  │ TiDB: Automatic data distribution                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  VITESS vs PROXYSQL:                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ProxySQL: Connection pooling, query routing        │    │
│  │ Vitess: Full sharding solution + management        │    │
│  │ Vitess: More comprehensive but more complex        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```
