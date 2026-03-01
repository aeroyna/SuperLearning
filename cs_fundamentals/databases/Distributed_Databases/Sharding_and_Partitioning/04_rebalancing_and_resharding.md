# Rebalancing and Resharding

## Why Rebalance?

```
┌─────────────────────────────────────────────────────────────┐
│                   Rebalancing Triggers                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  • Add new nodes (scale out)                                │
│  • Remove failed nodes                                       │
│  • Uneven data distribution (hotspots)                      │
│  • Uneven load distribution                                  │
│  • Hardware upgrade (bigger nodes)                          │
│                                                              │
│  Goals:                                                      │
│  • Minimize data movement                                    │
│  • Minimize downtime                                         │
│  • Maintain availability during rebalance                   │
│  • Ensure consistency                                        │
└─────────────────────────────────────────────────────────────┘
```

## Rebalancing Strategies

### Fixed Partitions

```
┌─────────────────────────────────────────────────────────────┐
│                   Fixed Partitions                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Create more partitions than nodes (e.g., 1000 partitions) │
│  Assign partitions to nodes                                  │
│                                                              │
│  3 nodes, 12 partitions:                                    │
│  N1: [P1, P4, P7, P10]                                      │
│  N2: [P2, P5, P8, P11]                                      │
│  N3: [P3, P6, P9, P12]                                      │
│                                                              │
│  Add N4:                                                     │
│  N1: [P1, P7, P10]    ← P4 moved                           │
│  N2: [P2, P8, P11]    ← P5 moved                           │
│  N3: [P3, P9, P12]    ← P6 moved                           │
│  N4: [P4, P5, P6]     ← Takes 3 partitions                 │
│                                                              │
│  PROS: Simple, move whole partitions                        │
│  CONS: Must choose partition count upfront                  │
│                                                              │
│  Used in: Elasticsearch, Kafka, Riak                       │
└─────────────────────────────────────────────────────────────┘
```

### Dynamic Partitioning

```
┌─────────────────────────────────────────────────────────────┐
│                  Dynamic Partitioning                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Split large partitions, merge small ones                   │
│                                                              │
│  Split:                                                      │
│  [──────────P1──────────]  (too big, >10GB)                │
│         ↓ split                                              │
│  [────P1a────][────P1b────]                                 │
│                                                              │
│  Merge:                                                      │
│  [──P1──][──P2──]  (both small, <1GB each)                 │
│         ↓ merge                                              │
│  [────────P1+P2────────]                                    │
│                                                              │
│  PROS: Adapts to data growth                                │
│  CONS: More complex, potential oscillation                  │
│                                                              │
│  Used in: HBase, MongoDB                                    │
└─────────────────────────────────────────────────────────────┘
```

## Online Resharding

```
┌─────────────────────────────────────────────────────────────┐
│                   Online Resharding                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Phase 1: Prepare                                            │
│  • Create new shard                                          │
│  • Set up replication from old shards                       │
│                                                              │
│  Phase 2: Copy (Background)                                 │
│  • Stream data to new shard                                  │
│  • Track changes during copy                                │
│  • Old shards continue serving traffic                      │
│                                                              │
│  Phase 3: Catch-up                                           │
│  • Apply buffered changes                                   │
│  • Get close to real-time                                    │
│                                                              │
│  Phase 4: Cutover                                            │
│  • Brief pause or dual-write                                │
│  • Update routing                                            │
│  • Verify consistency                                        │
│                                                              │
│  Phase 5: Cleanup                                            │
│  • Remove old replicas                                       │
│  • Update metadata                                           │
│                                                              │
│  Key: Never stop serving traffic                            │
└─────────────────────────────────────────────────────────────┘
```

## Best Practices

```
┌─────────────────────────────────────────────────────────────┐
│              Rebalancing Best Practices                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Over-partition from the start                           │
│     • Easier to move whole partitions                       │
│     • 10-100x expected node count                           │
│                                                              │
│  2. Use consistent hashing with vnodes                      │
│     • Smooth rebalancing                                     │
│     • Minimal data movement                                  │
│                                                              │
│  3. Limit rebalancing rate                                   │
│     • Don't saturate network                                │
│     • Keep serving traffic                                   │
│                                                              │
│  4. Monitor and alert                                        │
│     • Track partition sizes                                  │
│     • Detect hotspots early                                 │
│                                                              │
│  5. Test resharding procedures                              │
│     • Practice in staging                                    │
│     • Have rollback plan                                     │
└─────────────────────────────────────────────────────────────┘
```
