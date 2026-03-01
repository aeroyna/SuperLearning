# Database Selection

## Introduction

Choosing the right database is one of the most critical architectural decisions in software development. The wrong choice can lead to performance bottlenecks, scalability issues, and costly migrations. This section covers how to evaluate databases and make informed decisions.

## Topics in This Section

1. **[Choosing the Right Database](01_choosing_the_right_database.md)**
2. **[Polyglot Persistence](02_polyglot_persistence.md)**
3. **[Migration Between Databases](03_migration_between_databases.md)**

## Database Landscape Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Database Categories                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RELATIONAL (SQL)                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PostgreSQL │ MySQL │ SQL Server │ Oracle │ SQLite          │ │
│  │                                                             │ │
│  │ Best for: Structured data, ACID transactions, complex      │ │
│  │           queries, reporting, traditional applications     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DOCUMENT                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ MongoDB │ CouchDB │ Amazon DocumentDB │ Firebase Firestore │ │
│  │                                                             │ │
│  │ Best for: Flexible schemas, content management,            │ │
│  │           catalogs, user profiles, real-time apps          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  KEY-VALUE                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Redis │ Memcached │ Amazon DynamoDB │ etcd │ Riak          │ │
│  │                                                             │ │
│  │ Best for: Caching, sessions, real-time data,              │ │
│  │           leaderboards, simple lookups                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  WIDE-COLUMN                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Cassandra │ HBase │ ScyllaDB │ Google Bigtable             │ │
│  │                                                             │ │
│  │ Best for: Time-series, IoT, write-heavy workloads,        │ │
│  │           high availability at scale                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  GRAPH                                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Neo4j │ Amazon Neptune │ JanusGraph │ TigerGraph           │ │
│  │                                                             │ │
│  │ Best for: Social networks, recommendations,               │ │
│  │           fraud detection, knowledge graphs                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SEARCH                                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Elasticsearch │ Solr │ Meilisearch │ Typesense             │ │
│  │                                                             │ │
│  │ Best for: Full-text search, log analytics,                │ │
│  │           faceted search, autocomplete                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TIME-SERIES                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ InfluxDB │ TimescaleDB │ Prometheus │ QuestDB              │ │
│  │                                                             │ │
│  │ Best for: Metrics, monitoring, IoT sensor data,           │ │
│  │           financial tick data                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NEWSQL                                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CockroachDB │ TiDB │ Google Spanner │ YugabyteDB           │ │
│  │                                                             │ │
│  │ Best for: Global distribution with ACID,                  │ │
│  │           horizontal scaling with SQL                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Key Decision Factors

```
┌─────────────────────────────────────────────────────────────────┐
│                 Database Selection Criteria                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DATA MODEL                                                      │
│  • How is your data structured?                                 │
│  • How often does the schema change?                            │
│  • What are the relationships between entities?                 │
│                                                                  │
│  QUERY PATTERNS                                                  │
│  • Simple key lookups vs complex joins?                         │
│  • Read-heavy or write-heavy?                                   │
│  • Real-time or batch processing?                               │
│                                                                  │
│  SCALE REQUIREMENTS                                              │
│  • Expected data volume (GB, TB, PB)?                           │
│  • Requests per second?                                         │
│  • Horizontal vs vertical scaling needs?                        │
│                                                                  │
│  CONSISTENCY REQUIREMENTS                                        │
│  • Strong consistency (banking, inventory)?                     │
│  • Eventual consistency acceptable (social feeds)?              │
│  • ACID transactions needed?                                    │
│                                                                  │
│  OPERATIONAL CONSIDERATIONS                                      │
│  • Team expertise                                               │
│  • Managed vs self-hosted                                       │
│  • Cost (licensing, infrastructure, operations)                 │
│  • Ecosystem and tooling                                        │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Reference: When to Use What

```
┌─────────────────────────────────────────────────────────────────┐
│                    Quick Decision Guide                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "I need..."                          → Consider                │
│  ─────────────────────────────────────────────────────────────  │
│  ACID transactions + complex queries  → PostgreSQL, MySQL       │
│  Flexible schema + rapid development  → MongoDB                 │
│  Sub-millisecond caching              → Redis, Memcached        │
│  Massive write throughput             → Cassandra, ScyllaDB     │
│  Graph traversals                     → Neo4j, Neptune          │
│  Full-text search                     → Elasticsearch           │
│  Time-series metrics                  → TimescaleDB, InfluxDB   │
│  Global ACID at scale                 → CockroachDB, Spanner    │
│  Embedded database                    → SQLite, RocksDB         │
│  Message queue/streaming              → Redis Streams, Kafka    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
