# Leaderless Replication

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│              Leaderless Architecture                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  No single leader - any node can accept writes              │
│  Client writes to multiple nodes, reads from multiple       │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  Client ────────────────────────────────────►       │    │
│  │    │         Write to N=3 nodes                     │    │
│  │    ├──────────→ ┌──────┐                            │    │
│  │    │            │Node 1│ ✓                          │    │
│  │    │            └──────┘                            │    │
│  │    ├──────────→ ┌──────┐                            │    │
│  │    │            │Node 2│ ✓   W=2 acks needed       │    │
│  │    │            └──────┘                            │    │
│  │    └──────────→ ┌──────┐                            │    │
│  │                 │Node 3│ (slow/down)                │    │
│  │                 └──────┘                            │    │
│  │                                                      │    │
│  │  Success after W=2 acknowledgments                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Used in: Cassandra, DynamoDB, Riak, Voldemort            │
└─────────────────────────────────────────────────────────────┘
```

## Quorum

```
┌─────────────────────────────────────────────────────────────┐
│                    Quorum System                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  N = total replicas                                          │
│  W = write quorum (acks needed for write success)           │
│  R = read quorum (nodes to read from)                       │
│                                                              │
│  Consistency guarantee: W + R > N                           │
│                                                              │
│  Example (N=3):                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ W=2, R=2: Strong consistency (2+2=4 > 3)           │    │
│  │ W=1, R=3: Fast writes, slow reads                  │    │
│  │ W=3, R=1: Slow writes, fast reads                  │    │
│  │ W=1, R=1: Fast but eventually consistent (1+1=2 ≤3)│    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Read repair:                                                │
│  • Read from R nodes                                         │
│  • If values differ, use version with highest timestamp    │
│  • Write correct value back to stale nodes                 │
└─────────────────────────────────────────────────────────────┘
```

## Sloppy Quorum and Hinted Handoff

```
┌─────────────────────────────────────────────────────────────┐
│            Sloppy Quorum and Hinted Handoff                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Regular quorum: Must write to designated N nodes          │
│  If some are down: Write fails                              │
│                                                              │
│  Sloppy quorum: Write to ANY available W nodes             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Key K normally on: N1, N2, N3                       │    │
│  │ N1 is down                                           │    │
│  │                                                      │    │
│  │ Sloppy quorum allows:                               │    │
│  │ Write to N2, N3, N4 (N4 not normally for this key) │    │
│  │                                                      │    │
│  │ N4 stores a "hinted" value                          │    │
│  │ When N1 recovers: N4 hands off data to N1          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Trade-off:                                                  │
│  ✓ Higher availability during partitions                   │
│  ✗ W + R > N doesn't guarantee consistency                 │
│    (might read from different node set than written)       │
└─────────────────────────────────────────────────────────────┘
```
