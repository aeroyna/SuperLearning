# Sharding and Replication

## Learning Objectives
- Configure replica sets for high availability
- Implement horizontal scaling with sharding
- Choose appropriate shard keys
- Manage distributed MongoDB deployments

---

## 1. Replica Sets

### Replica Set Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MongoDB Replica Set                               │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                        Application                           │    │
│  │                            │                                 │    │
│  │                  ┌─────────┴─────────┐                      │    │
│  │                  ▼                   ▼                      │    │
│  │              Writes              Reads                      │    │
│  └──────────────────┼───────────────────┼──────────────────────┘    │
│                     │                   │                            │
│                     ▼                   ▼                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │   ┌──────────┐     ┌──────────┐     ┌──────────┐           │    │
│  │   │ PRIMARY  │────▶│SECONDARY │────▶│SECONDARY │           │    │
│  │   │  (Write) │◀────│  (Read)  │◀────│  (Read)  │           │    │
│  │   └──────────┘     └──────────┘     └──────────┘           │    │
│  │        │                 │                │                 │    │
│  │        └─────────────────┼────────────────┘                 │    │
│  │                    Replication                              │    │
│  │                      (oplog)                                │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Features:                                                           │
│  • Automatic failover (election of new primary)                     │
│  • Data redundancy across nodes                                     │
│  • Read scaling (read from secondaries)                             │
│  • No single point of failure                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Setting Up Replica Set

```javascript
// Start mongod with replica set name
// mongod --replSet "rs0" --port 27017 --dbpath /data/rs0-0

// Initialize replica set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongodb0.example.com:27017" },
    { _id: 1, host: "mongodb1.example.com:27017" },
    { _id: 2, host: "mongodb2.example.com:27017" }
  ]
})

// Check status
rs.status()

// Add member
rs.add("mongodb3.example.com:27017")

// Remove member
rs.remove("mongodb3.example.com:27017")

// Step down primary (force election)
rs.stepDown()

// Configuration
rs.conf()
rs.reconfig(newConfig)
```

### Member Types

```javascript
// Regular member (can become primary)
{ _id: 0, host: "mongodb0:27017" }

// Priority 0 (cannot become primary)
{ _id: 1, host: "mongodb1:27017", priority: 0 }

// Hidden member (not visible to clients)
{ _id: 2, host: "mongodb2:27017", hidden: true, priority: 0 }

// Delayed member (lags behind)
{
  _id: 3,
  host: "mongodb3:27017",
  priority: 0,
  hidden: true,
  secondaryDelaySecs: 3600  // 1 hour delay
}

// Arbiter (votes only, no data)
{ _id: 4, host: "mongodb4:27017", arbiterOnly: true }
```

### Read Preferences

```javascript
// Read from primary (default)
db.collection.find().readPref("primary")

// Read from primary if available
db.collection.find().readPref("primaryPreferred")

// Read from secondary
db.collection.find().readPref("secondary")

// Read from secondary if available
db.collection.find().readPref("secondaryPreferred")

// Read from nearest (lowest latency)
db.collection.find().readPref("nearest")

// With tag set
db.collection.find().readPref("secondary", [{ region: "us-east" }])
```

### Write Concern

```javascript
// Acknowledge after primary write
db.collection.insertOne({ x: 1 }, { writeConcern: { w: 1 } })

// Acknowledge after majority
db.collection.insertOne({ x: 1 }, { writeConcern: { w: "majority" } })

// Acknowledge after all members
db.collection.insertOne({ x: 1 }, { writeConcern: { w: 3 } })

// With journal
db.collection.insertOne({ x: 1 }, {
  writeConcern: { w: "majority", j: true }
})

// With timeout
db.collection.insertOne({ x: 1 }, {
  writeConcern: { w: "majority", wtimeout: 5000 }
})
```

---

## 2. Sharding

### Sharded Cluster Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Sharded Cluster                                   │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Application                             │    │
│  │                           │                                  │    │
│  │                           ▼                                  │    │
│  │                    ┌───────────┐                             │    │
│  │                    │  mongos   │  Query Router               │    │
│  │                    │  (Router) │                             │    │
│  │                    └─────┬─────┘                             │    │
│  └──────────────────────────┼──────────────────────────────────┘    │
│                             │                                        │
│    ┌────────────────────────┼────────────────────────┐              │
│    │                        │                        │              │
│    ▼                        ▼                        ▼              │
│  ┌──────────┐         ┌──────────┐         ┌──────────┐            │
│  │  Shard 1 │         │  Shard 2 │         │  Shard 3 │            │
│  │ (rs0)    │         │ (rs1)    │         │ (rs2)    │            │
│  │┌────────┐│         │┌────────┐│         │┌────────┐│            │
│  ││Primary ││         ││Primary ││         ││Primary ││            │
│  │├────────┤│         │├────────┤│         │├────────┤│            │
│  ││Second. ││         ││Second. ││         ││Second. ││            │
│  │├────────┤│         │├────────┤│         │├────────┤│            │
│  ││Second. ││         ││Second. ││         ││Second. ││            │
│  │└────────┘│         │└────────┘│         │└────────┘│            │
│  └──────────┘         └──────────┘         └──────────┘            │
│                                                                      │
│              ┌────────────────────────────┐                         │
│              │      Config Servers        │                         │
│              │  (Cluster Metadata)        │                         │
│              │  ┌────┐ ┌────┐ ┌────┐     │                         │
│              │  │cfg0│ │cfg1│ │cfg2│     │                         │
│              │  └────┘ └────┘ └────┘     │                         │
│              └────────────────────────────┘                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Setting Up Sharding

```javascript
// 1. Start config servers (replica set)
// mongod --configsvr --replSet configRS --port 27019

// 2. Start shard servers (each a replica set)
// mongod --shardsvr --replSet shard0 --port 27018

// 3. Start mongos router
// mongos --configdb configRS/cfg0:27019,cfg1:27019,cfg2:27019

// 4. Connect to mongos and add shards
sh.addShard("shard0/mongodb0:27018,mongodb1:27018,mongodb2:27018")
sh.addShard("shard1/mongodb3:27018,mongodb4:27018,mongodb5:27018")

// 5. Enable sharding for database
sh.enableSharding("mydb")

// 6. Shard a collection
sh.shardCollection("mydb.orders", { customerId: 1 })
```

### Shard Keys

```javascript
// Ranged shard key
sh.shardCollection("mydb.orders", { orderDate: 1 })

// Hashed shard key (even distribution)
sh.shardCollection("mydb.users", { _id: "hashed" })

// Compound shard key
sh.shardCollection("mydb.logs", { tenantId: 1, timestamp: 1 })

// Shard key properties:
// - Immutable (cannot change after sharding)
// - Must be indexed
// - Affects query routing and data distribution
```

### Shard Key Selection

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Shard Key Considerations                          │
│                                                                      │
│  GOOD Shard Key Properties:                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • High cardinality (many unique values)                     │    │
│  │ • Low frequency (values not repeated too often)             │    │
│  │ • Non-monotonic (avoids hot spots)                          │    │
│  │ • Query isolation (queries target single shard)             │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  BAD Shard Keys:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Monotonically increasing (ObjectId, timestamp)            │    │
│  │   → All writes go to one shard                               │    │
│  │ • Low cardinality (status, country)                          │    │
│  │   → Uneven distribution, jumbo chunks                        │    │
│  │ • Frequently updated fields                                  │    │
│  │   → Cannot update shard key                                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Examples:                                                           │
│  ✓ { customerId: 1 } - Good if many customers                       │
│  ✓ { customerId: 1, orderDate: 1 } - Compound for better isolation  │
│  ✓ { _id: "hashed" } - Good for write distribution                  │
│  ✗ { createdAt: 1 } - Hot spot on recent shard                      │
│  ✗ { status: 1 } - Low cardinality                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Query Routing

```javascript
// Targeted query (includes shard key)
db.orders.find({ customerId: 123 })
// → Routed to single shard

// Scatter-gather (no shard key)
db.orders.find({ status: "pending" })
// → Query all shards, merge results

// Broadcast operations
db.orders.createIndex({ status: 1 })
// → Runs on all shards
```

---

## 3. Sharding Operations

### Chunks and Balancing

```javascript
// View chunk distribution
db.orders.getShardDistribution()

// View all chunks
use config
db.chunks.find({ ns: "mydb.orders" }).pretty()

// Manual chunk split
sh.splitAt("mydb.orders", { customerId: 500000 })

// Move chunk to specific shard
sh.moveChunk("mydb.orders", { customerId: 100000 }, "shard2")

// Balancer status
sh.getBalancerState()
sh.isBalancerRunning()

// Stop/start balancer
sh.stopBalancer()
sh.startBalancer()

// Schedule balancer window
use config
db.settings.updateOne(
  { _id: "balancer" },
  {
    $set: {
      activeWindow: { start: "02:00", stop: "06:00" }
    }
  },
  { upsert: true }
)
```

### Zone Sharding

```javascript
// Create zones for geographic distribution
sh.addShardTag("shard0", "US")
sh.addShardTag("shard1", "EU")
sh.addShardTag("shard2", "APAC")

// Define zone ranges
sh.addTagRange(
  "mydb.users",
  { region: "US", _id: MinKey },
  { region: "US", _id: MaxKey },
  "US"
)

sh.addTagRange(
  "mydb.users",
  { region: "EU", _id: MinKey },
  { region: "EU", _id: MaxKey },
  "EU"
)

// View zones
sh.status()
```

---

## 4. Monitoring and Maintenance

### Replica Set Monitoring

```javascript
// Replica set status
rs.status()

// Check replication lag
rs.printSecondaryReplicationInfo()

// Oplog status
rs.printReplicationInfo()

// Member health
db.adminCommand({ replSetGetStatus: 1 })
```

### Sharded Cluster Monitoring

```javascript
// Cluster status
sh.status()

// Shard distribution
db.collection.getShardDistribution()

// Config database info
use config
db.shards.find()
db.databases.find()
db.chunks.find({ ns: "mydb.orders" })

// Current operations
db.currentOp()

// Server statistics
db.serverStatus()
```

### Common Issues

```javascript
// Check for jumbo chunks (too large to move)
use config
db.chunks.find({ jumbo: true })

// Clear jumbo flag after splitting
db.chunks.updateOne(
  { _id: chunkId },
  { $unset: { jumbo: 1 } }
)

// Check primary shard
use config
db.databases.find({ _id: "mydb" })

// Change primary shard
db.adminCommand({ movePrimary: "mydb", to: "shard2" })
```

---

## 5. Best Practices

### Replica Sets

```
✓ Use odd number of voting members (3, 5, 7)
✓ Distribute members across data centers
✓ Configure proper write concern for durability
✓ Monitor replication lag
✓ Use hidden members for backups
✓ Test failover regularly
```

### Sharding

```
✓ Choose shard key carefully (cannot change easily)
✓ Use compound shard keys when appropriate
✓ Pre-split data for initial load
✓ Monitor chunk distribution
✓ Schedule balancing during off-peak
✓ Plan for resharding complexity
```

### Connection Strings

```javascript
// Replica set connection
"mongodb://host1:27017,host2:27017,host3:27017/mydb?replicaSet=rs0"

// Sharded cluster connection (mongos)
"mongodb://mongos1:27017,mongos2:27017/mydb"

// With options
"mongodb://host1,host2,host3/mydb?replicaSet=rs0&readPreference=secondaryPreferred&w=majority"
```

---

## Summary

| Feature | Replica Set | Sharding |
|---------|-------------|----------|
| Purpose | High availability | Horizontal scaling |
| Data | Full copy on each node | Partitioned across shards |
| Writes | Primary only | Distributed by shard key |
| Reads | Any member | Any shard (with routing) |
| Failover | Automatic election | Per-shard failover |

---

## Further Reading

- MongoDB Replication documentation
- MongoDB Sharding documentation
- Production Deployment Best Practices
