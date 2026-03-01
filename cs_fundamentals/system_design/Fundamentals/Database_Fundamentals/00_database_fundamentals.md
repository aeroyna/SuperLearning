# Database Fundamentals

Understanding database internals and trade-offs is essential for system design. This section covers the key concepts for choosing and optimizing databases.

---

## Core Decisions

When designing a system, you need to answer:
1. **SQL or NoSQL?** - Structured vs flexible schema
2. **How to index?** - Optimize for your query patterns
3. **How to scale?** - Sharding, replication, or both

---

## Topics in This Section

- [3.1 SQL vs NoSQL](01_sql_vs_nosql.md)
- [3.2 Database Indexing](02_database_indexing.md)
- [3.3 Database Sharding](03_database_sharding.md)
- [3.4 Database Partitioning](04_database_partitioning.md)

---

## Quick Reference: Database Selection

| Requirement | Database Type | Examples |
|-------------|--------------|----------|
| ACID transactions | SQL | PostgreSQL, MySQL |
| Flexible schema | Document | MongoDB, CouchDB |
| High write throughput | Wide-column | Cassandra, HBase |
| Simple key-value | Key-Value | Redis, DynamoDB |
| Relationships | Graph | Neo4j, Amazon Neptune |
| Full-text search | Search Engine | Elasticsearch, Solr |
| Time-series data | Time-Series | InfluxDB, TimescaleDB |

---

## Database Properties: ACID vs BASE

### ACID (Traditional SQL)
- **Atomicity**: Transaction is all-or-nothing
- **Consistency**: Database moves from valid state to valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data survives failures

### BASE (Many NoSQL)
- **Basically Available**: System is always available
- **Soft state**: State may change over time (without input)
- **Eventually consistent**: System becomes consistent over time

---

## Interview Quick Reference

```
When asked "What database would you use?":

1. Identify data characteristics:
   - Structured or unstructured?
   - Read-heavy or write-heavy?
   - Relationships between entities?
   - Scale requirements?

2. Consider consistency requirements:
   - Need ACID? → SQL
   - Eventually consistent OK? → NoSQL options open

3. Consider access patterns:
   - Key-value lookups? → Redis, DynamoDB
   - Complex queries with joins? → PostgreSQL, MySQL
   - Full-text search? → Elasticsearch

4. State your choice with justification
```
