# Redis - In-Memory Data Store

## Overview

Redis (Remote Dictionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. Its exceptional speed and versatile data structures make it essential for high-performance applications.

---

## Why Redis?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Value Proposition                          │
│                                                                      │
│  Performance:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Sub-millisecond latency (typically < 1ms)                 │    │
│  │ • 100,000+ operations per second (single instance)          │    │
│  │ • All data in memory for instant access                     │    │
│  │ • Single-threaded event loop (no lock contention)           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Versatility:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Rich data structures (not just key-value)                 │    │
│  │ • Pub/Sub messaging                                         │    │
│  │ • Streams for event sourcing                                │    │
│  │ • Lua scripting for atomic operations                       │    │
│  │ • Transactions with MULTI/EXEC                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Common Use Cases:                                                   │
│  • Session storage           • Real-time analytics                  │
│  • Caching                   • Message queues                       │
│  • Rate limiting             • Leaderboards                         │
│  • Distributed locks         • Geospatial indexes                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Redis Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Server Architecture                         │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Client Connections                      │    │
│  │   ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐         │    │
│  │   │ C1 │ │ C2 │ │ C3 │ │ C4 │ │ C5 │ │ C6 │ │ Cn │         │    │
│  │   └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘         │    │
│  └──────┼──────┼──────┼──────┼──────┼──────┼──────┼────────────┘    │
│         └──────┴──────┴──────┼──────┴──────┴──────┘                 │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   Event Loop (Single Thread)                 │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │    │
│  │  │   Accept    │  │    Read     │  │   Write     │          │    │
│  │  │ Connections │  │  Commands   │  │  Responses  │          │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    In-Memory Database                        │    │
│  │                                                              │    │
│  │   ┌─────────────────────────────────────────────────────┐   │    │
│  │   │                  Key Space (Dict)                    │   │    │
│  │   │                                                      │   │    │
│  │   │  key1 ──► String   key4 ──► Set                     │   │    │
│  │   │  key2 ──► List     key5 ──► Sorted Set              │   │    │
│  │   │  key3 ──► Hash     key6 ──► Stream                  │   │    │
│  │   │                                                      │   │    │
│  │   └─────────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                    ┌─────────┴─────────┐                            │
│                    ▼                   ▼                            │
│  ┌──────────────────────┐  ┌──────────────────────┐                 │
│  │    RDB Snapshots     │  │    AOF (Append Only) │                 │
│  │  (Point-in-time)     │  │    (Write log)       │                 │
│  └──────────────────────┘  └──────────────────────┘                 │
│                              │                                       │
│                              ▼                                       │
│                         ┌────────┐                                  │
│                         │  Disk  │                                  │
│                         └────────┘                                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Data Types at a Glance

| Type | Description | Key Commands |
|------|-------------|--------------|
| **String** | Binary-safe strings, up to 512MB | GET, SET, INCR, APPEND |
| **List** | Linked list of strings | LPUSH, RPUSH, LPOP, LRANGE |
| **Set** | Unordered unique strings | SADD, SMEMBERS, SINTER |
| **Sorted Set** | Scored unique strings | ZADD, ZRANGE, ZRANGEBYSCORE |
| **Hash** | Field-value pairs | HSET, HGET, HGETALL |
| **Stream** | Append-only log | XADD, XREAD, XRANGE |
| **Bitmap** | Bit operations on strings | SETBIT, GETBIT, BITCOUNT |
| **HyperLogLog** | Probabilistic cardinality | PFADD, PFCOUNT |
| **Geospatial** | Coordinates and radius queries | GEOADD, GEORADIUS |

---

## Basic Operations

```redis
# Strings
SET user:1:name "John Doe"
GET user:1:name                 # "John Doe"
INCR page:views                 # Atomic increment
SETEX session:abc 3600 "data"   # Set with 1 hour TTL

# Keys
KEYS user:*                     # Find matching keys (don't use in prod)
SCAN 0 MATCH user:* COUNT 100   # Cursor-based iteration
TTL session:abc                 # Time to live
EXPIRE key 60                   # Set expiration
DEL key1 key2                   # Delete keys
EXISTS key                      # Check existence

# Transactions
MULTI                           # Start transaction
SET foo 1
INCR foo
EXEC                            # Execute atomically

# Pipelining (client-side batching)
# Send multiple commands without waiting for replies
```

---

## Redis vs Other Databases

```
┌─────────────────────────────────────────────────────────────────────┐
│                    When to Choose Redis                             │
│                                                                      │
│  Choose Redis when you need:                                         │
│  ✓ Sub-millisecond latency                                          │
│  ✓ High throughput (100K+ ops/sec)                                  │
│  ✓ Rich data structure operations                                   │
│  ✓ Pub/Sub messaging                                                │
│  ✓ Automatic key expiration                                         │
│                                                                      │
│  Consider alternatives when:                                         │
│  • Data exceeds available RAM                                        │
│  • Complex queries with joins needed                                 │
│  • Strong durability is critical                                     │
│  • ACID transactions across multiple keys required                  │
│                                                                      │
│  Comparison:                                                         │
│  ┌──────────────┬───────────────┬───────────────┬────────────────┐  │
│  │              │    Redis      │   Memcached   │   PostgreSQL   │  │
│  ├──────────────┼───────────────┼───────────────┼────────────────┤  │
│  │ Latency      │ < 1ms         │ < 1ms         │ 1-10ms         │  │
│  │ Data Types   │ Rich          │ Strings only  │ SQL types      │  │
│  │ Persistence  │ Optional      │ No            │ Yes            │  │
│  │ Clustering   │ Yes           │ Client-side   │ Read replicas  │  │
│  │ Pub/Sub      │ Yes           │ No            │ LISTEN/NOTIFY  │  │
│  │ Scripting    │ Lua           │ No            │ PL/pgSQL       │  │
│  └──────────────┴───────────────┴───────────────┴────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Section Contents

### 20.1 Data Structures
Deep dive into Redis data types: strings, lists, sets, sorted sets, hashes, and specialized types like HyperLogLog and Bitmaps.

### 20.2 Persistence Options
Configure RDB snapshots and AOF logging for durability, with trade-offs between performance and data safety.

### 20.3 Pub/Sub and Streams
Real-time messaging with Pub/Sub and event streaming with Redis Streams for event sourcing and message queues.

### 20.4 Clustering and Sentinel
High availability with Redis Sentinel and horizontal scaling with Redis Cluster for distributed deployments.

### 20.5 Caching Patterns
Cache-aside, write-through, write-behind patterns with TTL strategies and cache invalidation techniques.

### 20.6 Lua Scripting
Atomic operations with Lua scripts, reducing network round-trips and implementing complex logic server-side.

---

## Quick Start

```bash
# Install Redis
# Ubuntu/Debian
sudo apt update && sudo apt install redis-server

# macOS
brew install redis

# Docker
docker run -d --name redis -p 6379:6379 redis:latest

# Start Redis CLI
redis-cli

# Basic test
127.0.0.1:6379> PING
PONG

127.0.0.1:6379> SET greeting "Hello, Redis!"
OK

127.0.0.1:6379> GET greeting
"Hello, Redis!"
```

---

## Learning Path

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Learning Progression                        │
│                                                                      │
│  Beginner:                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │ Basic       │  │ Data        │  │ Key         │                  │
│  │ Commands    │──▶│ Structures  │──▶│ Expiration  │                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
│                                                                      │
│  Intermediate:                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │ Persistence │  │ Pub/Sub     │  │ Transactions│                  │
│  │ RDB/AOF     │──▶│ Messaging   │──▶│ & Pipelining│                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
│                                                                      │
│  Advanced:                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │ Lua         │  │ Cluster     │  │ Streams &   │                  │
│  │ Scripting   │──▶│ & Sentinel  │──▶│ Event Source│                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Speed**: In-memory storage with O(1) operations for most commands
2. **Versatility**: Rich data structures beyond simple key-value
3. **Simplicity**: Easy to learn, single-threaded model
4. **Flexibility**: Optional persistence, replication, clustering
5. **Ecosystem**: Widely supported across all programming languages
