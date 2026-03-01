# Sharding and Partitioning

## Introduction

Sharding (horizontal partitioning) distributes data across multiple nodes to achieve scalability beyond a single machine's capacity. Each shard contains a subset of the data, enabling parallel processing and distributing load.

## Topics in This Section

1. **[Horizontal vs Vertical Partitioning](01_horizontal_vs_vertical_partitioning.md)**
2. **[Sharding Strategies](02_sharding_strategies.md)**
3. **[Consistent Hashing](03_consistent_hashing.md)**
4. **[Rebalancing and Resharding](04_rebalancing_and_resharding.md)**

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│                  Partitioning Types                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  HORIZONTAL (Sharding):                                      │
│  Different rows on different nodes                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │ Users   │  │ Users   │  │ Users   │                     │
│  │ A-H     │  │ I-P     │  │ Q-Z     │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
│   Shard 1      Shard 2      Shard 3                        │
│                                                              │
│  VERTICAL:                                                   │
│  Different columns on different nodes                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │ Users   │  │ Users   │  │ Users   │                     │
│  │id,name  │  │email,pwd│  │profile  │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
│                                                              │
│  FUNCTIONAL:                                                 │
│  Different tables/domains on different nodes                │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │ Users   │  │ Orders  │  │Products │                     │
│  │ Service │  │ Service │  │ Service │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

## Key Concepts

### Partition Key Selection

```
Good partition keys:
• High cardinality (many distinct values)
• Even distribution
• Matches query patterns
• Doesn't create hotspots

Examples:
• User ID for user data
• Order ID for orders
• Timestamp + random for time-series (avoid hot partition)
```

### Shard Mapping

```
How to find which shard has the data:

HASH-BASED:        shard = hash(key) mod num_shards
RANGE-BASED:       shard = lookup(key_range → shard)
DIRECTORY-BASED:   shard = directory.lookup(key)
CONSISTENT HASH:   shard = ring.find(hash(key))
```
