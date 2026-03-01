# Deployment and Operations

## Learning Objectives
- Configure Neo4j for production environments
- Implement clustering for high availability
- Optimize performance and memory
- Monitor and maintain Neo4j deployments

---

## 1. Installation and Configuration

### Installation Options

```bash
# Docker (quickest)
docker run -d \
    --name neo4j \
    -p 7474:7474 \
    -p 7687:7687 \
    -v $HOME/neo4j/data:/data \
    -v $HOME/neo4j/logs:/logs \
    -v $HOME/neo4j/plugins:/plugins \
    -e NEO4J_AUTH=neo4j/password123 \
    -e NEO4J_PLUGINS='["apoc", "graph-data-science"]' \
    neo4j:5-enterprise

# Linux package
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -
echo 'deb https://debian.neo4j.com stable latest' | sudo tee /etc/apt/sources.list.d/neo4j.list
sudo apt update && sudo apt install neo4j

# Start service
sudo systemctl start neo4j
sudo systemctl enable neo4j
```

### Configuration Files

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Configuration Locations                           │
│                                                                      │
│  /etc/neo4j/                    (package install)                   │
│  /var/lib/neo4j/conf/           (alternative)                       │
│  $NEO4J_HOME/conf/              (tarball install)                   │
│                                                                      │
│  Key Files:                                                          │
│  ├── neo4j.conf         Main configuration                         │
│  ├── apoc.conf          APOC plugin settings                       │
│  └── neo4j-admin.conf   Admin tool settings                        │
│                                                                      │
│  Data Locations:                                                     │
│  /var/lib/neo4j/data/                                               │
│  ├── databases/         Database files                              │
│  ├── transactions/      Transaction logs                            │
│  └── dbms/              System database                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Configuration Settings

```properties
# neo4j.conf

# Network
server.default_listen_address=0.0.0.0
server.bolt.listen_address=:7687
server.http.listen_address=:7474
server.https.listen_address=:7473

# Security
dbms.security.auth_enabled=true
dbms.security.procedures.unrestricted=apoc.*,gds.*

# Memory (crucial for performance)
server.memory.heap.initial_size=4g
server.memory.heap.max_size=4g
server.memory.pagecache.size=2g

# Transaction logs
db.tx_log.rotation.retention_policy=2 days
db.tx_log.rotation.size=256M

# Query logging
db.logs.query.enabled=INFO
db.logs.query.threshold=5s

# Directories
server.directories.data=/var/lib/neo4j/data
server.directories.logs=/var/log/neo4j
server.directories.import=/var/lib/neo4j/import
```

---

## 2. Memory Configuration

### Memory Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Neo4j Memory Model                                │
│                                                                      │
│  Total Available RAM: 32 GB                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  ┌────────────────────────────────────────────────────────┐ │    │
│  │  │           JVM Heap (8-16 GB)                           │ │    │
│  │  │  • Query execution                                     │ │    │
│  │  │  • Transaction state                                   │ │    │
│  │  │  • Cypher planning                                     │ │    │
│  │  └────────────────────────────────────────────────────────┘ │    │
│  │                                                              │    │
│  │  ┌────────────────────────────────────────────────────────┐ │    │
│  │  │           Page Cache (10-20 GB)                        │ │    │
│  │  │  • Graph data cache                                    │ │    │
│  │  │  • Node/relationship storage                           │ │    │
│  │  │  • Property storage                                    │ │    │
│  │  │  • Index data                                          │ │    │
│  │  └────────────────────────────────────────────────────────┘ │    │
│  │                                                              │    │
│  │  ┌────────────────────────────────────────────────────────┐ │    │
│  │  │           OS Reserve (2-4 GB)                          │ │    │
│  │  │  • File system cache                                   │ │    │
│  │  │  • OS operations                                       │ │    │
│  │  └────────────────────────────────────────────────────────┘ │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Memory Recommendations

```bash
# Get memory recommendation
neo4j-admin server memory-recommendation

# Output example:
# Based on the current database size, the recommended settings are:
#   server.memory.heap.initial_size=8g
#   server.memory.heap.max_size=8g
#   server.memory.pagecache.size=12g
```

```properties
# neo4j.conf

# For 32GB server with 50GB database:
server.memory.heap.initial_size=8g
server.memory.heap.max_size=8g
server.memory.pagecache.size=12g

# For 64GB server with 200GB database:
server.memory.heap.initial_size=16g
server.memory.heap.max_size=16g
server.memory.pagecache.size=40g

# Rule of thumb:
# Page cache = 1.2 * database size (ideal: entire DB in cache)
# Heap = 8-16GB (rarely need more)
# Leave 2-4GB for OS
```

---

## 3. Clustering

### Cluster Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Neo4j Cluster (Causal Clustering)                 │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                         Cluster                              │    │
│  │                                                              │    │
│  │   Primary Members (Core Servers):                           │    │
│  │   ┌─────────┐     ┌─────────┐     ┌─────────┐              │    │
│  │   │  Core1  │◀───▶│  Core2  │◀───▶│  Core3  │              │    │
│  │   │ LEADER  │     │FOLLOWER │     │FOLLOWER │              │    │
│  │   └────┬────┘     └────┬────┘     └────┬────┘              │    │
│  │        │               │               │                    │    │
│  │        └───────────────┴───────────────┘                    │    │
│  │                        │                                    │    │
│  │                   Raft Protocol                             │    │
│  │                  (Consensus)                                │    │
│  │                        │                                    │    │
│  │   Secondary Members (Read Replicas):                        │    │
│  │   ┌─────────┐     ┌─────────┐     ┌─────────┐              │    │
│  │   │ Replica1│     │ Replica2│     │ Replica3│              │    │
│  │   │  (Read) │     │  (Read) │     │  (Read) │              │    │
│  │   └─────────┘     └─────────┘     └─────────┘              │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Core Servers:                                                       │
│  • Handle writes (through leader)                                   │
│  • Participate in consensus                                         │
│  • Store full copy of data                                          │
│  • Minimum: 3 for fault tolerance                                   │
│                                                                      │
│  Read Replicas:                                                      │
│  • Handle read queries only                                         │
│  • Eventually consistent                                            │
│  • Scale read capacity                                              │
└─────────────────────────────────────────────────────────────────────┘
```

### Cluster Configuration

```properties
# neo4j.conf (Core Server 1)

# Cluster mode
dbms.mode=CORE

# Server identity
server.default_advertised_address=core1.example.com

# Discovery
causal_clustering.discovery_type=DNS
causal_clustering.initial_discovery_members=core1.example.com:5000,core2.example.com:5000,core3.example.com:5000

# Cluster communication ports
causal_clustering.discovery_listen_address=:5000
causal_clustering.transaction_listen_address=:6000
causal_clustering.raft_listen_address=:7000

# Minimum core servers
causal_clustering.minimum_core_cluster_size_at_formation=3
```

```properties
# neo4j.conf (Read Replica)

# Cluster mode
dbms.mode=READ_REPLICA

# Discovery
causal_clustering.discovery_type=DNS
causal_clustering.initial_discovery_members=core1.example.com:5000,core2.example.com:5000,core3.example.com:5000
```

### Cluster Operations

```bash
# Check cluster status
CALL dbms.cluster.overview()

# Role information
CALL dbms.cluster.role()

# Routing table
CALL dbms.routing.getRoutingTable({}, "neo4j")

# Force leader election (careful!)
CALL dbms.cluster.forceLeaderElection()
```

---

## 4. Security

### Authentication

```properties
# neo4j.conf

# Enable authentication
dbms.security.auth_enabled=true

# Native auth provider
dbms.security.authentication_providers=native
dbms.security.authorization_providers=native

# LDAP (optional)
# dbms.security.authentication_providers=ldap,native
# dbms.security.ldap.host=ldap://ldap.example.com
# dbms.security.ldap.authentication.user_dn_template=uid={0},ou=users,dc=example,dc=com
```

```cypher
// User management
CREATE USER alice SET PASSWORD 'secret' CHANGE NOT REQUIRED
ALTER USER alice SET PASSWORD 'newsecret'
DROP USER alice

// Role management
CREATE ROLE analyst
GRANT ROLE analyst TO alice

// Privileges
GRANT MATCH {*} ON GRAPH * TO analyst
GRANT READ {*} ON GRAPH * TO analyst
REVOKE CREATE ON GRAPH * FROM analyst

// Show grants
SHOW USER alice PRIVILEGES
```

### SSL/TLS

```properties
# neo4j.conf

# Enable HTTPS
server.https.enabled=true

# SSL certificates
dbms.ssl.policy.bolt.enabled=true
dbms.ssl.policy.bolt.base_directory=/var/lib/neo4j/certificates/bolt
dbms.ssl.policy.bolt.private_key=private.key
dbms.ssl.policy.bolt.public_certificate=public.crt
dbms.ssl.policy.bolt.client_auth=NONE

# Force encrypted connections
dbms.connector.bolt.tls_level=REQUIRED
```

---

## 5. Backup and Recovery

### Backup

```bash
# Online backup (Enterprise)
neo4j-admin database backup neo4j --to-path=/backup/

# Backup specific database
neo4j-admin database backup neo4j --to-path=/backup/ --include-metadata=all

# Dump database (offline)
neo4j-admin database dump neo4j --to-path=/backup/neo4j.dump

# Backup with compression
neo4j-admin database backup neo4j \
    --to-path=/backup/ \
    --compress

# Incremental backup
neo4j-admin database backup neo4j \
    --to-path=/backup/ \
    --incremental-from=/backup/previous/
```

### Restore

```bash
# Restore from backup
neo4j-admin database restore \
    --from-path=/backup/neo4j-2024-01-15/ \
    --database=neo4j \
    --force

# Load from dump
neo4j-admin database load neo4j \
    --from-path=/backup/neo4j.dump \
    --overwrite-destination

# After restore
neo4j-admin database migrate neo4j

# Start database
cypher-shell -u neo4j -p password "START DATABASE neo4j"
```

### Backup Strategy

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Backup Best Practices                             │
│                                                                      │
│  Full Backup:                                                        │
│  • Weekly or after major changes                                    │
│  • Store off-site                                                   │
│  • Test restores regularly                                          │
│                                                                      │
│  Incremental Backup:                                                 │
│  • Daily or more frequent                                           │
│  • Based on transaction logs                                        │
│  • Faster than full backup                                          │
│                                                                      │
│  Retention:                                                          │
│  • Keep 30 days of incrementals                                     │
│  • Keep 3 months of full backups                                    │
│  • Keep 1 year of monthly backups                                   │
│                                                                      │
│  Automation:                                                         │
│  • Cron job for scheduled backups                                   │
│  • Monitor backup success/failure                                   │
│  • Alert on backup failures                                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Performance Tuning

### Indexes

```cypher
// Create index for fast lookup
CREATE INDEX person_name FOR (p:Person) ON (p.name)

// Composite index
CREATE INDEX person_name_age FOR (p:Person) ON (p.name, p.age)

// Text index (full-text search)
CREATE TEXT INDEX person_bio FOR (p:Person) ON (p.bio)

// Relationship index
CREATE INDEX knows_since FOR ()-[k:KNOWS]-() ON (k.since)

// Check index usage
EXPLAIN MATCH (p:Person {name: 'Alice'}) RETURN p
```

### Query Optimization

```cypher
// Use PROFILE to analyze queries
PROFILE MATCH (p:Person)-[:KNOWS*1..5]->(friend)
WHERE p.name = 'Alice'
RETURN friend

// Use EXPLAIN for query plan
EXPLAIN MATCH (p:Person)-[:KNOWS]->(f) RETURN p, f

// Avoid cartesian products
// Bad:
MATCH (a:Person), (b:Product) RETURN a, b

// Good:
MATCH (a:Person)-[:PURCHASED]->(b:Product) RETURN a, b

// Use parameters for query caching
MATCH (p:Person {name: $name}) RETURN p

// Limit variable-length paths
MATCH path = (a)-[:KNOWS*1..3]->(b)  // Not *
RETURN path
LIMIT 100
```

### Transaction Management

```cypher
// Large operations in batches
CALL apoc.periodic.iterate(
    "MATCH (p:Person) WHERE p.needsUpdate RETURN p",
    "SET p.updated = true",
    {batchSize: 1000, parallel: true}
)

// Batch import
LOAD CSV WITH HEADERS FROM 'file:///data.csv' AS row
CALL {
    WITH row
    CREATE (n:Node {id: row.id, name: row.name})
} IN TRANSACTIONS OF 1000 ROWS
```

---

## 7. Monitoring

### Built-in Monitoring

```cypher
// Current transactions
SHOW TRANSACTIONS

// Kill long-running query
TERMINATE TRANSACTION "neo4j-transaction-123"

// Database info
SHOW DATABASES

// Memory info
CALL dbms.listPools()

// Query statistics
CALL db.stats.retrieve('QUERIES')
```

### JMX Metrics

```properties
# neo4j.conf
# Enable JMX
metrics.enabled=true
metrics.jmx.enabled=true
metrics.csv.enabled=true
metrics.csv.interval=30s
metrics.csv.path=/var/lib/neo4j/metrics
```

### Key Metrics to Monitor

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Monitoring Checklist                              │
│                                                                      │
│  Performance:                                                        │
│  • Query execution time (p99, p999)                                 │
│  • Transaction commit rate                                          │
│  • Page cache hit ratio (should be > 99%)                           │
│  • GC pause times                                                   │
│                                                                      │
│  Resources:                                                          │
│  • Heap usage                                                       │
│  • Page cache usage                                                 │
│  • Disk I/O                                                         │
│  • CPU utilization                                                  │
│                                                                      │
│  Cluster (if applicable):                                            │
│  • Cluster member status                                            │
│  • Replication lag                                                  │
│  • Leader elections                                                 │
│                                                                      │
│  Database:                                                           │
│  • Store size                                                       │
│  • Node/relationship count                                          │
│  • Active connections                                               │
│  • Open transactions                                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Prometheus Integration

```yaml
# docker-compose.yml
services:
  neo4j:
    image: neo4j:5-enterprise
    environment:
      - NEO4J_PLUGINS=["apoc"]
      - NEO4J_metrics_prometheus_enabled=true
      - NEO4J_metrics_prometheus_endpoint=localhost:2004
    ports:
      - "2004:2004"

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

---

## 8. Maintenance

### Database Maintenance

```bash
# Check consistency
neo4j-admin database check neo4j

# Compact database (reclaim space)
neo4j-admin database copy neo4j compacted-neo4j --compact-node-store

# Migration
neo4j-admin database migrate neo4j
```

### Log Management

```properties
# neo4j.conf

# Query logging
db.logs.query.enabled=INFO
db.logs.query.threshold=5s
db.logs.query.parameter_logging_enabled=true

# Log rotation
server.logs.gc.rotation.size=50M
server.logs.gc.rotation.keep_number=5
```

### Health Checks

```cypher
// Basic health check
RETURN 1

// Database status
SHOW DATABASES

// Cluster health
CALL dbms.cluster.overview()
YIELD id, addresses, role, groups, database
RETURN *
```

---

## Summary

| Area | Key Points |
|------|------------|
| Memory | Page cache + Heap + OS reserve |
| Cluster | 3+ cores for HA, read replicas for scaling |
| Security | Auth + SSL + Role-based access |
| Backup | Regular + tested + off-site |
| Performance | Indexes + Query optimization + Batching |

---

## Best Practices

```
Installation:
✓ Use enterprise edition for production
✓ Separate data and log directories
✓ Use SSDs for data storage
✓ Configure memory appropriately

Clustering:
✓ Use odd number of core servers (3, 5, 7)
✓ Add read replicas for read scaling
✓ Monitor replication lag
✓ Test failover procedures

Security:
✓ Enable authentication
✓ Use SSL/TLS in production
✓ Implement least-privilege access
✓ Regular security audits

Operations:
✓ Automate backups
✓ Monitor key metrics
✓ Plan maintenance windows
✓ Keep software updated
```
