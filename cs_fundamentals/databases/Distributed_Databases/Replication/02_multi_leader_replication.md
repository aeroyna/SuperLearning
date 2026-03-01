# Multi-Leader Replication

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│              Multi-Leader Architecture                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Multiple nodes can accept writes                           │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  DC East           DC West           DC Europe       │    │
│  │  ┌────────┐       ┌────────┐       ┌────────┐       │    │
│  │  │ Leader │ ←───→ │ Leader │ ←───→ │ Leader │       │    │
│  │  │   1    │       │   2    │       │   3    │       │    │
│  │  └───┬────┘       └───┬────┘       └───┬────┘       │    │
│  │      │                │                │             │    │
│  │  ┌───▼───┐        ┌───▼───┐        ┌───▼───┐        │    │
│  │  │Replica│        │Replica│        │Replica│        │    │
│  │  └───────┘        └───────┘        └───────┘        │    │
│  │                                                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use cases:                                                  │
│  • Multi-datacenter deployment                              │
│  • Offline operation (mobile/laptop)                        │
│  • Collaborative editing                                     │
└─────────────────────────────────────────────────────────────┘
```

## Conflict Handling

```
┌─────────────────────────────────────────────────────────────┐
│                   Conflict Scenarios                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Write-Write Conflict:                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Leader 1: UPDATE users SET name='Alice'             │    │
│  │ Leader 2: UPDATE users SET name='Bob'  (same row)  │    │
│  │                                                      │    │
│  │ Both succeed locally → Conflict when syncing       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Resolution strategies:                                      │
│  • Conflict avoidance: Route same data to same leader      │
│  • Last-write-wins (LWW): Use timestamps                   │
│  • Merge: Combine changes (CRDTs)                          │
│  • Custom: Application-specific logic                       │
│  • Keep all: Return all versions to application            │
└─────────────────────────────────────────────────────────────┘
```

## Topology

```
┌─────────────────────────────────────────────────────────────┐
│                Replication Topologies                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ALL-TO-ALL:                                                 │
│  ┌───┐ ──→ ┌───┐                                           │
│  │ L1│ ←── │ L2│  Every leader replicates to every other   │
│  └───┘ ←─→ └───┘                                           │
│    ↕   ↗↙   ↕                                               │
│  ┌───┐ ──→ ┌───┐                                           │
│  │ L3│ ←── │ L4│  ✓ Fault tolerant, ✗ Complex              │
│  └───┘     └───┘                                            │
│                                                              │
│  CIRCULAR:                                                   │
│  L1 → L2 → L3 → L4 → L1                                    │
│  ✓ Simple, ✗ Single failure breaks chain                   │
│                                                              │
│  STAR:                                                       │
│     L2                                                       │
│      ↑                                                       │
│  L1←─C─→L3   Central node relays                           │
│      ↓                                                       │
│     L4                                                       │
│  ✓ Simple, ✗ Central point of failure                      │
└─────────────────────────────────────────────────────────────┘
```
