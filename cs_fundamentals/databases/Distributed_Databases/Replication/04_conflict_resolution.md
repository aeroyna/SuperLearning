# Conflict Resolution

## Types of Conflicts

```
┌─────────────────────────────────────────────────────────────┐
│                    Conflict Types                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  WRITE-WRITE:                                                │
│  Two clients update same data concurrently                  │
│  A: balance = 100    B: balance = 200                       │
│                                                              │
│  DELETE-UPDATE:                                              │
│  One client deletes, another updates                        │
│  A: DELETE user      B: UPDATE user SET name='X'            │
│                                                              │
│  UNIQUENESS:                                                 │
│  Both insert with same unique key                           │
│  A: INSERT (id=1)    B: INSERT (id=1)                       │
└─────────────────────────────────────────────────────────────┘
```

## Resolution Strategies

```
┌─────────────────────────────────────────────────────────────┐
│              Resolution Strategies                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  LAST-WRITE-WINS (LWW):                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Pick value with highest timestamp                   │    │
│  │ A: (value=100, ts=1000)                             │    │
│  │ B: (value=200, ts=1005)                             │    │
│  │ Winner: B (ts=1005 > ts=1000)                       │    │
│  │                                                      │    │
│  │ ⚠ Loses A's write silently                         │    │
│  │ ⚠ Clock skew can cause issues                      │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used in: Cassandra, DynamoDB                              │
│                                                              │
│  VERSION VECTORS:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Track version per node                               │    │
│  │ A: {A:1, B:0}  →  value=100                         │    │
│  │ B: {A:0, B:1}  →  value=200                         │    │
│  │                                                      │    │
│  │ Neither dominates → concurrent conflict detected   │    │
│  │ Return both to application for resolution          │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used in: Riak, DynamoDB (optional)                        │
│                                                              │
│  CRDTs (Conflict-free Replicated Data Types):              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Data structures that always merge correctly         │    │
│  │ Counter: sum of all increments                      │    │
│  │ Set: union of all additions                         │    │
│  │ No conflicts by design                              │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used in: Riak, Redis (CRDT module)                        │
│                                                              │
│  APPLICATION RESOLUTION:                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Return all versions to application                  │    │
│  │ App decides based on business logic                 │    │
│  │ Example: Merge shopping cart items                  │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used in: CouchDB, custom implementations                  │
└─────────────────────────────────────────────────────────────┘
```

## Best Practices

```
┌─────────────────────────────────────────────────────────────┐
│            Conflict Resolution Best Practices                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. AVOID CONFLICTS:                                         │
│     • Route same data to same node (affinity)              │
│     • Use optimistic locking (version checks)              │
│     • Design for append-only when possible                  │
│                                                              │
│  2. USE APPROPRIATE STRATEGY:                               │
│     • LWW: When losing updates is acceptable               │
│     • CRDTs: For counters, sets, registers                 │
│     • Version vectors: When you need to detect conflicts  │
│                                                              │
│  3. TEST CONFLICT SCENARIOS:                                 │
│     • Simulate network partitions                           │
│     • Test resolution in staging                            │
│     • Monitor for conflict rates                            │
└─────────────────────────────────────────────────────────────┘
```
