# Sharding Strategies

## Range-Based Sharding

```
┌─────────────────────────────────────────────────────────────┐
│                   Range-Based Sharding                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Partition by key ranges:                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Shard 1: A-H  │ Shard 2: I-P  │ Shard 3: Q-Z       │    │
│  │ Alice, Bob,   │ Ivan, Jane,   │ Quinn, Rachel,     │    │
│  │ Carol, ...    │ Kim, ...      │ Steve, ...         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ADVANTAGES:                                                 │
│  • Range queries efficient (single shard or few)           │
│  • Sorted data within shards                                │
│  • Easy to understand                                        │
│                                                              │
│  DISADVANTAGES:                                              │
│  • Hotspots if keys are sequential (timestamps)            │
│  • Uneven distribution if key distribution skewed          │
│  • Rebalancing may require moving large ranges             │
│                                                              │
│  USE CASES:                                                  │
│  • Time-series (partition by month/year)                   │
│  • Alphabetical data                                         │
│  • Geographic (partition by region)                         │
└─────────────────────────────────────────────────────────────┘
```

## Hash-Based Sharding

```
┌─────────────────────────────────────────────────────────────┐
│                   Hash-Based Sharding                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  shard = hash(key) mod num_shards                           │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ hash("alice") = 742  → 742 mod 3 = 1 → Shard 1     │    │
│  │ hash("bob")   = 891  → 891 mod 3 = 0 → Shard 0     │    │
│  │ hash("carol") = 156  → 156 mod 3 = 0 → Shard 0     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ADVANTAGES:                                                 │
│  • Even distribution (with good hash function)             │
│  • No hotspots from sequential keys                         │
│  • Simple to implement                                       │
│                                                              │
│  DISADVANTAGES:                                              │
│  • Range queries require scatter-gather                     │
│  • Changing shard count = massive data movement            │
│  • All keys may need to move                                │
│                                                              │
│  Problem with mod N:                                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ N=3: hash=742 → shard 1                             │    │
│  │ N=4: hash=742 → shard 2  ← Different shard!        │    │
│  │                                                      │    │
│  │ Adding one shard moves ~75% of keys                 │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Directory-Based Sharding

```
┌─────────────────────────────────────────────────────────────┐
│                  Directory-Based Sharding                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Lookup service maps keys to shards:                        │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           Directory Service                          │    │
│  │  ┌──────────────────────────────────────────────┐   │    │
│  │  │ Key        │ Shard                           │   │    │
│  │  │ "alice"    │ shard-2                         │   │    │
│  │  │ "bob"      │ shard-1                         │   │    │
│  │  │ "tenant-A" │ shard-5                         │   │    │
│  │  └──────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────┘    │
│                      ↓ Lookup                               │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │ Shard 1 │  │ Shard 2 │  │ Shard 5 │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
│                                                              │
│  ADVANTAGES:                                                 │
│  • Flexible placement                                        │
│  • Easy to move individual keys                             │
│  • Can implement any sharding logic                         │
│                                                              │
│  DISADVANTAGES:                                              │
│  • Directory is single point of failure                    │
│  • Extra hop for every query                                │
│  • Directory must be highly available                       │
└─────────────────────────────────────────────────────────────┘
```

## Compound Sharding

```
┌─────────────────────────────────────────────────────────────┐
│                   Compound Sharding                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Multiple levels of sharding:                                │
│                                                              │
│  Level 1: By tenant (directory)                             │
│  Level 2: By hash within tenant                             │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Tenant A (Small): Shards 1-3                        │    │
│  │ Tenant B (Large): Shards 4-10                       │    │
│  │ Tenant C (Medium): Shards 11-15                     │    │
│  │                                                      │    │
│  │ Within each tenant: hash(user_id) mod tenant_shards│    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Benefits:                                                   │
│  • Isolate tenants for performance/compliance              │
│  • Scale individual tenants independently                  │
│  • Combine range and hash benefits                          │
└─────────────────────────────────────────────────────────────┘
```
