# Apache Cassandra - Distributed Wide-Column Store

## Overview

Apache Cassandra is a highly scalable, distributed NoSQL database designed for handling large amounts of data across many commodity servers with no single point of failure. Originally developed at Facebook, it combines the distributed architecture of Amazon Dynamo with the data model of Google Bigtable.

---

## Why Cassandra?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cassandra Value Proposition                       │
│                                                                      │
│  Scale:                                                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Linear scalability (add nodes = add capacity)             │    │
│  │ • Handles petabytes of data                                 │    │
│  │ • Millions of operations per second                         │    │
│  │ • Proven at Apple (400K+ nodes), Netflix, Instagram        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Availability:                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • No single point of failure (masterless architecture)     │    │
│  │ • Multi-datacenter replication                              │    │
│  │ • Continues operating during node failures                  │    │
│  │ • Tunable consistency levels                                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Performance:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Optimized for write-heavy workloads                       │    │
│  │ • Consistent single-digit millisecond latency               │    │
│  │ • Append-only storage (no read-before-write)                │    │
│  │ • Compaction for read optimization                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Common Use Cases:                                                   │
│  • Time-series data           • IoT sensor data                     │
│  • Messaging systems          • User activity tracking              │
│  • Recommendation engines     • Fraud detection                     │
│  • Product catalogs           • Real-time analytics                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cassandra Ring Architecture                       │
│                                                                      │
│                         Token Ring                                   │
│                                                                      │
│                          ┌─────┐                                     │
│                      ┌───│Node1│───┐                                 │
│                     ╱    └─────┘    ╲                                │
│                ┌─────┐            ┌─────┐                            │
│               │Node6│              │Node2│                            │
│                └─────┘            └─────┘                            │
│                    │                │                                 │
│                    │   Gossip       │                                 │
│                    │   Protocol     │                                 │
│                    │                │                                 │
│                ┌─────┐            ┌─────┐                            │
│               │Node5│              │Node3│                            │
│                └─────┘            └─────┘                            │
│                     ╲    ┌─────┐    ╱                                │
│                      └───│Node4│───┘                                 │
│                          └─────┘                                     │
│                                                                      │
│  Key Concepts:                                                       │
│  • All nodes are equal (peer-to-peer)                               │
│  • Data distributed by partition key hash                           │
│  • Each node responsible for token range                            │
│  • Replication factor determines copies                             │
│  • Gossip protocol for cluster state                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Data Model

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cassandra Data Model                              │
│                                                                      │
│  Keyspace (like database)                                           │
│  └── Table (column family)                                          │
│      └── Partition (group of rows)                                  │
│          └── Row (unique by clustering key)                         │
│              └── Column (name-value pair)                           │
│                                                                      │
│  Example Table: user_events                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Partition Key: user_id                                       │    │
│  │ Clustering Key: event_timestamp DESC                         │    │
│  │                                                              │    │
│  │ user_id │ event_timestamp    │ event_type │ data            │    │
│  │─────────┼────────────────────┼────────────┼─────────────────│    │
│  │ user1   │ 2024-01-15 10:30   │ login      │ {ip: ...}       │    │
│  │ user1   │ 2024-01-15 10:15   │ click      │ {page: ...}     │    │
│  │ user1   │ 2024-01-15 10:00   │ login      │ {ip: ...}       │    │
│  │─────────┼────────────────────┼────────────┼─────────────────│    │
│  │ user2   │ 2024-01-15 11:00   │ purchase   │ {item: ...}     │    │
│  │ user2   │ 2024-01-15 10:45   │ click      │ {page: ...}     │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  • Partition key determines data location                           │
│  • Clustering key determines sort order within partition            │
│  • Wide rows: millions of columns per partition                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Cassandra vs Other Databases

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Database Comparison                               │
│                                                                      │
│  Feature         │ Cassandra │ MongoDB  │ PostgreSQL │ Redis       │
│  ────────────────┼───────────┼──────────┼────────────┼─────────────│
│  Data Model      │ Wide-column│ Document │ Relational │ Key-Value  │
│  Scaling         │ Horizontal│ Horizontal│ Vertical  │ Horizontal │
│  Consistency     │ Tunable   │ Strong   │ Strong     │ Eventual   │
│  Query Language  │ CQL       │ MQL      │ SQL        │ Commands   │
│  Joins           │ No        │ Limited  │ Yes        │ No         │
│  Schema          │ Flexible  │ Flexible │ Fixed      │ Schemaless │
│  Best For        │ Write-heavy│ Documents│ ACID      │ Cache      │
│                                                                      │
│  Choose Cassandra when:                                              │
│  ✓ Write-heavy workloads (100K+ writes/sec)                         │
│  ✓ Linear scalability required                                      │
│  ✓ Always-on availability critical                                  │
│  ✓ Time-series or event data                                        │
│  ✓ Multi-datacenter replication needed                              │
│                                                                      │
│  Consider alternatives when:                                         │
│  • Complex queries with joins                                        │
│  • Strong consistency required everywhere                            │
│  • Small dataset (< 100GB)                                           │
│  • Ad-hoc query patterns                                             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Section Contents

### 21.1 Data Modeling
Design tables for query patterns, understand partition and clustering keys, avoid common modeling mistakes.

### 21.2 CQL and Operations
Master Cassandra Query Language, CRUD operations, batch statements, and lightweight transactions.

### 21.3 Partitioning Strategy
Choose effective partition keys, size partitions correctly, and understand data distribution.

### 21.4 Consistency Tuning
Configure consistency levels, understand read/write paths, and balance availability vs consistency.

### 21.5 Cluster Operations
Manage nodes, perform repairs, handle compaction, and monitor cluster health.

---

## Quick Start

```bash
# Install Cassandra
# Ubuntu/Debian
echo "deb https://debian.cassandra.apache.org 41x main" | sudo tee /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt update && sudo apt install cassandra

# Docker
docker run -d --name cassandra -p 9042:9042 cassandra:4.1

# Connect with cqlsh
cqlsh

# Create keyspace
CREATE KEYSPACE IF NOT EXISTS my_app
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 3
};

USE my_app;

# Create table
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  name TEXT,
  email TEXT,
  created_at TIMESTAMP
);

# Insert data
INSERT INTO users (user_id, name, email, created_at)
VALUES (uuid(), 'John Doe', 'john@example.com', toTimestamp(now()));

# Query data
SELECT * FROM users WHERE user_id = <uuid>;
```

---

## Key Terminology

| Term | Definition |
|------|------------|
| **Keyspace** | Top-level container (like database), defines replication |
| **Table** | Collection of rows with defined schema |
| **Partition Key** | Determines which node stores the data |
| **Clustering Key** | Determines sort order within partition |
| **Primary Key** | Partition key + clustering key(s) |
| **Replication Factor** | Number of copies of each partition |
| **Consistency Level** | How many replicas must respond |
| **Gossip** | Protocol for cluster state communication |
| **Compaction** | Process of merging SSTables |
| **SSTable** | Sorted String Table (immutable on-disk files) |

---

## Learning Path

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cassandra Learning Progression                    │
│                                                                      │
│  Beginner:                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │ CQL Basics  │  │ Data Types  │  │ Basic       │                  │
│  │ & CRUD      │──▶│ & Tables    │──▶│ Modeling    │                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
│                                                                      │
│  Intermediate:                                                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │ Partitioning│  │ Consistency │  │ Secondary   │                  │
│  │ Strategy    │──▶│ Tuning      │──▶│ Indexes     │                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
│                                                                      │
│  Advanced:                                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │
│  │ Cluster     │  │ Performance │  │ Multi-DC    │                  │
│  │ Operations  │──▶│ Tuning      │──▶│ Deployment  │                  │
│  └─────────────┘  └─────────────┘  └─────────────┘                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **Masterless**: No single point of failure, all nodes are equal
2. **Write-optimized**: Append-only storage model, excellent write performance
3. **Partition-centric**: Data modeling driven by partition key design
4. **Tunable consistency**: Balance between availability and consistency
5. **Linear scalability**: Add nodes to increase capacity linearly
