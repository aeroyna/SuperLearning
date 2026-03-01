# Replication

## Introduction

Replication maintains copies of data on multiple nodes for fault tolerance, scalability, and reduced latency. Different replication strategies offer different trade-offs between consistency, availability, and performance.

## Topics in This Section

1. **[Single-Leader Replication](01_single_leader_replication.md)**
2. **[Multi-Leader Replication](02_multi_leader_replication.md)**
3. **[Leaderless Replication](03_leaderless_replication.md)**
4. **[Conflict Resolution](04_conflict_resolution.md)**

## Replication Architectures Overview

```
┌─────────────────────────────────────────────────────────────┐
│              Replication Architectures                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  SINGLE-LEADER (Primary-Secondary):                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │         ┌─────────┐                                 │    │
│  │  Writes │ Primary │ Reads                          │    │
│  │    ───→ │ (Leader)│ ←───                           │    │
│  │         └────┬────┘                                 │    │
│  │         ┌────┴────┐                                 │    │
│  │    ┌────▼───┐ ┌───▼────┐                           │    │
│  │    │Replica │ │Replica │  Reads only               │    │
│  │    └────────┘ └────────┘                           │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used: PostgreSQL, MySQL, MongoDB                          │
│                                                              │
│  MULTI-LEADER (Multi-Primary):                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  ┌─────────┐    ┌─────────┐                        │    │
│  │  │Primary 1│←──→│Primary 2│ ← Both accept writes   │    │
│  │  └─────────┘    └─────────┘                        │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used: CouchDB, multi-DC setups                            │
│                                                              │
│  LEADERLESS:                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  ┌─────┐   ┌─────┐   ┌─────┐                       │    │
│  │  │Node │   │Node │   │Node │ ← All accept writes  │    │
│  │  └─────┘   └─────┘   └─────┘                       │    │
│  │  Quorum: W + R > N                                 │    │
│  └─────────────────────────────────────────────────────┘    │
│  Used: Cassandra, DynamoDB, Riak                           │
└─────────────────────────────────────────────────────────────┘
```

## Synchronous vs Asynchronous

```
┌─────────────────────────────────────────────────────────────┐
│           Sync vs Async Replication                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  SYNCHRONOUS:                                                │
│  Client → Primary → Wait for replica ack → Client OK       │
│  ✓ No data loss on primary failure                         │
│  ✗ Higher latency, availability tied to replica            │
│                                                              │
│  ASYNCHRONOUS:                                               │
│  Client → Primary → Client OK → Replicate later            │
│  ✓ Lower latency                                            │
│  ✗ Possible data loss if primary fails before replication  │
│                                                              │
│  SEMI-SYNCHRONOUS:                                           │
│  Wait for at least 1 replica, others async                  │
│  Balance between durability and latency                     │
└─────────────────────────────────────────────────────────────┘
```
