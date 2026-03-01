# Horizontal vs Vertical Partitioning

## Horizontal Partitioning (Sharding)

```
┌─────────────────────────────────────────────────────────────┐
│             Horizontal Partitioning                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Original Table (1M rows):                                   │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ id  │ name    │ email           │ region  │ ...      │  │
│  │ 1   │ Alice   │ alice@mail.com  │ US      │ ...      │  │
│  │ 2   │ Bob     │ bob@mail.com    │ EU      │ ...      │  │
│  │ ... │ ...     │ ...             │ ...     │ ...      │  │
│  │ 1M  │ Zoe     │ zoe@mail.com    │ ASIA    │ ...      │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  After Sharding by Region:                                   │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐│
│  │ Shard US        │ │ Shard EU        │ │ Shard ASIA      ││
│  │ 400K rows       │ │ 350K rows       │ │ 250K rows       ││
│  │ id,name,email...│ │ id,name,email...│ │ id,name,email...││
│  └─────────────────┘ └─────────────────┘ └─────────────────┘│
│                                                              │
│  ✓ Each shard has complete rows (all columns)              │
│  ✓ Queries for single region hit one shard                 │
│  ✗ Cross-region queries need scatter-gather                │
└─────────────────────────────────────────────────────────────┘
```

### Advantages and Disadvantages

```
ADVANTAGES:
• Scale writes linearly by adding shards
• Each shard fits in memory/storage
• Isolate workloads by tenant/region
• Parallel query execution

DISADVANTAGES:
• Cross-shard queries expensive
• Transactions across shards complex
• Resharding is difficult
• Application complexity
```

## Vertical Partitioning

```
┌─────────────────────────────────────────────────────────────┐
│              Vertical Partitioning                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Original Table:                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ id │ name │ email │ password │ profile_json │ avatar │  │
│  │ 1  │ ...  │ ...   │ ...      │ (10KB)       │ (1MB)  │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                              │
│  After Vertical Partitioning:                                │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐│
│  │ Core Data       │ │ Profile Data    │ │ Media Store     ││
│  │ id,name,email   │ │ id,profile_json │ │ id,avatar       ││
│  │ password        │ │ (document DB)   │ │ (blob storage)  ││
│  │ (hot, indexed)  │ │ (moderate)      │ │ (cold, CDN)     ││
│  └─────────────────┘ └─────────────────┘ └─────────────────┘│
│                                                              │
│  ✓ Hot columns cached efficiently                          │
│  ✓ Right storage for each data type                        │
│  ✗ JOINs require cross-store queries                       │
└─────────────────────────────────────────────────────────────┘
```

## Choosing Between Them

```
┌─────────────────────────────────────────────────────────────┐
│                 Decision Matrix                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Use Horizontal When:                                        │
│  • Single table too large for one node                      │
│  • Need write scalability                                    │
│  • Clear partition key (tenant, region, time)               │
│  • Queries mostly within partition boundaries               │
│                                                              │
│  Use Vertical When:                                          │
│  • Different access patterns for different columns          │
│  • Some columns rarely accessed (cold data)                 │
│  • Different storage requirements (BLOB, document)          │
│  • Want to optimize cache efficiency                        │
│                                                              │
│  Use Both:                                                   │
│  • Large-scale systems often combine both                   │
│  • Vertical for column groups, horizontal for rows          │
└─────────────────────────────────────────────────────────────┘
```
