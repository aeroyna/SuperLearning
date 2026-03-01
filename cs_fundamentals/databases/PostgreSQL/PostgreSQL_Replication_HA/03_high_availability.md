# High Availability Patterns

## Learning Objectives
- Understand HA architecture patterns
- Implement automatic failover with Patroni
- Configure connection pooling with PgBouncer
- Design multi-region deployments

---

## 1. HA Architecture Patterns

### Single Primary with Standbys

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Basic HA Pattern                                  │
│                                                                      │
│                    ┌─────────────────┐                              │
│                    │  Load Balancer  │                              │
│                    │    (HAProxy)    │                              │
│                    └────────┬────────┘                              │
│                             │                                        │
│              ┌──────────────┼──────────────┐                        │
│              │              │              │                        │
│              ▼              ▼              ▼                        │
│       ┌──────────┐   ┌──────────┐   ┌──────────┐                   │
│       │  Primary │   │ Standby  │   │ Standby  │                   │
│       │   (RW)   │──▶│   (RO)   │──▶│   (RO)   │                   │
│       └──────────┘   └──────────┘   └──────────┘                   │
│              │              │              │                        │
│              └──────────────┼──────────────┘                        │
│                             │                                        │
│                    ┌────────▼────────┐                              │
│                    │  HA Manager     │                              │
│                    │  (Patroni)      │                              │
│                    └─────────────────┘                              │
│                                                                      │
│  Components:                                                         │
│  • Load balancer routes traffic                                     │
│  • Primary handles writes                                           │
│  • Standbys handle reads                                            │
│  • HA manager handles failover                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Components

```
HA Stack Components:

1. PostgreSQL (Primary + Standbys)
   • Streaming replication
   • Synchronous for durability

2. HA Manager (Patroni, pg_auto_failover)
   • Leader election
   • Automatic failover
   • Health monitoring

3. Distributed Consensus (etcd, Consul, ZooKeeper)
   • Stores cluster state
   • Prevents split-brain
   • Enables leader election

4. Connection Pooler (PgBouncer, Pgpool-II)
   • Connection multiplexing
   • Reduces connection overhead
   • Query routing

5. Load Balancer (HAProxy, pgpool)
   • Read/write splitting
   • Health checks
   • Failover redirection
```

---

## 2. Patroni

### What is Patroni?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Patroni Architecture                              │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    etcd/Consul/ZooKeeper                     │    │
│  │                    (Distributed Config Store)                │    │
│  │                    - Leader key                              │    │
│  │                    - Cluster topology                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│         ▲                    ▲                    ▲                  │
│         │                    │                    │                  │
│  ┌──────┴──────┐      ┌──────┴──────┐      ┌──────┴──────┐         │
│  │   Patroni   │      │   Patroni   │      │   Patroni   │         │
│  │   (Agent)   │      │   (Agent)   │      │   (Agent)   │         │
│  ├─────────────┤      ├─────────────┤      ├─────────────┤         │
│  │ PostgreSQL  │◄────▶│ PostgreSQL  │◄────▶│ PostgreSQL  │         │
│  │  (Leader)   │      │ (Replica)   │      │ (Replica)   │         │
│  └─────────────┘      └─────────────┘      └─────────────┘         │
│       Node 1               Node 2               Node 3              │
│                                                                      │
│  Patroni responsibilities:                                          │
│  • Manages PostgreSQL lifecycle                                     │
│  • Participates in leader election                                  │
│  • Handles failover/switchover                                      │
│  • Provides REST API for monitoring                                 │
└─────────────────────────────────────────────────────────────────────┘
```

### Patroni Configuration

```yaml
# patroni.yml
scope: postgres-cluster
name: node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: node1.example.com:8008

etcd3:
  hosts:
    - etcd1.example.com:2379
    - etcd2.example.com:2379
    - etcd3.example.com:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576  # 1MB
    synchronous_mode: true
    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
        wal_level: replica
        hot_standby: on
        max_connections: 200
        max_wal_senders: 10
        max_replication_slots: 10
        synchronous_commit: on
        synchronous_standby_names: '*'

  initdb:
    - encoding: UTF8
    - data-checksums

  pg_hba:
    - host replication replicator 0.0.0.0/0 scram-sha-256
    - host all all 0.0.0.0/0 scram-sha-256

  users:
    admin:
      password: admin_password
      options:
        - createrole
        - createdb
    replicator:
      password: rep_password
      options:
        - replication

postgresql:
  listen: 0.0.0.0:5432
  connect_address: node1.example.com:5432
  data_dir: /var/lib/postgresql/15/main
  bin_dir: /usr/lib/postgresql/15/bin
  authentication:
    replication:
      username: replicator
      password: rep_password
    superuser:
      username: postgres
      password: postgres_password
```

### Patroni Commands

```bash
# Cluster status
patronictl -c /etc/patroni/patroni.yml list

# Switchover (planned)
patronictl -c /etc/patroni/patroni.yml switchover
# Interactive: select new leader, current leader

# Failover (forced)
patronictl -c /etc/patroni/patroni.yml failover

# Restart PostgreSQL
patronictl -c /etc/patroni/patroni.yml restart postgres-cluster

# Reload configuration
patronictl -c /etc/patroni/patroni.yml reload postgres-cluster

# Edit dynamic configuration
patronictl -c /etc/patroni/patroni.yml edit-config

# Reinitialize replica
patronictl -c /etc/patroni/patroni.yml reinit postgres-cluster node2
```

### Patroni REST API

```bash
# Get cluster members
curl -s http://node1:8008/cluster | jq

# Check if leader
curl -s http://node1:8008/leader

# Health check endpoints
curl -s http://node1:8008/health      # Overall health
curl -s http://node1:8008/primary     # Returns 200 if primary
curl -s http://node1:8008/replica     # Returns 200 if replica
curl -s http://node1:8008/read-only   # Returns 200 if can serve reads
```

---

## 3. PgBouncer

### Connection Pooling Concept

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Why Connection Pooling?                           │
│                                                                      │
│  Without Pooler:                                                     │
│  ┌─────────────────┐        ┌─────────────────┐                     │
│  │   100 App       │        │   PostgreSQL    │                     │
│  │   Connections   │───────▶│   100 backends  │                     │
│  │   (idle most    │        │   (~100MB RAM)  │                     │
│  │    of time)     │        │                 │                     │
│  └─────────────────┘        └─────────────────┘                     │
│                                                                      │
│  With Pooler:                                                        │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────┐ │
│  │   100 App       │        │   PgBouncer     │        │  Postgres│ │
│  │   Connections   │───────▶│   (multiplexes) │───────▶│ 20 backs │ │
│  │                 │        │                 │        │  (~20MB) │ │
│  └─────────────────┘        └─────────────────┘        └─────────┘ │
│                                                                      │
│  Benefits:                                                           │
│  • Lower PostgreSQL memory usage                                    │
│  • Faster connection acquisition                                    │
│  • More application connections supported                           │
│  • Connection reuse across requests                                 │
└─────────────────────────────────────────────────────────────────────┘
```

### PgBouncer Configuration

```ini
# pgbouncer.ini

[databases]
# Database routing
mydb = host=primary.example.com port=5432 dbname=mydb
mydb_ro = host=replica1.example.com,replica2.example.com port=5432 dbname=mydb

# With auth_user for dynamic user lookup
* = host=primary.example.com port=5432 auth_user=pgbouncer

[pgbouncer]
# Listening
listen_addr = *
listen_port = 6432

# Authentication
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt

# Pooling mode
pool_mode = transaction  # session, transaction, statement

# Pool sizing
default_pool_size = 20
min_pool_size = 5
reserve_pool_size = 5
max_client_conn = 1000
max_db_connections = 50

# Timeouts
server_idle_timeout = 600
client_idle_timeout = 0
client_login_timeout = 60

# Logging
log_connections = 1
log_disconnections = 1
log_pooler_errors = 1

# Admin
admin_users = admin
stats_users = stats

# Application name tracking
application_name_add_host = 1
```

### Pool Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PgBouncer Pool Modes                              │
│                                                                      │
│  SESSION MODE:                                                       │
│  • Server assigned for entire client session                        │
│  • Most compatible (supports all features)                          │
│  • Least efficient (no sharing between sessions)                    │
│  • Use when: prepared statements, session variables needed          │
│                                                                      │
│  TRANSACTION MODE (recommended):                                     │
│  • Server assigned per transaction                                  │
│  • Good balance of compatibility and efficiency                     │
│  • Session features reset between transactions                      │
│  • Use when: Most web applications                                  │
│                                                                      │
│  STATEMENT MODE:                                                     │
│  • Server assigned per statement                                    │
│  • Most efficient but most restrictive                              │
│  • Multi-statement transactions don't work                          │
│  • Use when: Simple single-statement queries                        │
└─────────────────────────────────────────────────────────────────────┘
```

### PgBouncer Admin Console

```sql
-- Connect to admin database
psql -p 6432 -U admin pgbouncer

-- Show pools
SHOW POOLS;

-- Show stats
SHOW STATS;

-- Show servers
SHOW SERVERS;

-- Show clients
SHOW CLIENTS;

-- Show databases
SHOW DATABASES;

-- Pause/resume
PAUSE mydb;
RESUME mydb;

-- Reload config
RELOAD;

-- Disconnect clients
KILL mydb;
```

---

## 4. HAProxy Configuration

### Read/Write Splitting

```
# haproxy.cfg

global
    log /dev/log local0
    maxconn 4096

defaults
    mode tcp
    log global
    option tcplog
    option dontlognull
    timeout connect 5s
    timeout client 30m
    timeout server 30m

# Primary (read-write)
frontend pg_write
    bind *:5000
    default_backend pg_primary

backend pg_primary
    option httpchk GET /primary
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 node1:5432 check port 8008
    server node2 node2:5432 check port 8008
    server node3 node3:5432 check port 8008

# Replicas (read-only)
frontend pg_read
    bind *:5001
    default_backend pg_replicas

backend pg_replicas
    balance roundrobin
    option httpchk GET /replica
    http-check expect status 200
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions
    server node1 node1:5432 check port 8008
    server node2 node2:5432 check port 8008
    server node3 node3:5432 check port 8008

# Stats page
frontend stats
    bind *:7000
    mode http
    stats enable
    stats uri /stats
    stats refresh 10s
```

### Health Checks with Patroni

```
HAProxy uses Patroni REST API for health checks:
• GET /primary → 200 if primary, 503 otherwise
• GET /replica → 200 if replica, 503 otherwise
• GET /read-only → 200 if can serve reads

This ensures:
• Writes only go to current primary
• Reads distributed across all healthy nodes
• Failed nodes automatically removed
• Newly promoted primary gets traffic immediately
```

---

## 5. pg_auto_failover

### Alternative to Patroni

```
┌─────────────────────────────────────────────────────────────────────┐
│                    pg_auto_failover Architecture                     │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Monitor Node                              │    │
│  │                    (State Machine)                           │    │
│  │                    - Tracks node states                      │    │
│  │                    - Coordinates failover                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                ▲                          ▲                         │
│                │                          │                         │
│         ┌──────┴──────┐           ┌──────┴──────┐                  │
│         │   Primary   │           │   Secondary  │                  │
│         │   (pg_auto  │◄─────────▶│   (pg_auto   │                  │
│         │   failover) │           │   failover)  │                  │
│         └─────────────┘           └─────────────┘                   │
│                                                                      │
│  Simpler than Patroni:                                              │
│  • No external DCS (etcd/Consul) required                          │
│  • Monitor node provides consensus                                  │
│  • Easier to set up for small clusters                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Setup

```bash
# On monitor node
pg_autoctl create monitor \
    --pgdata /var/lib/pgsql/monitor \
    --pgport 5000 \
    --auth scram-sha-256

# On primary node
pg_autoctl create postgres \
    --pgdata /var/lib/pgsql/data \
    --pgport 5432 \
    --pgctl /usr/bin/pg_ctl \
    --monitor postgres://autoctl@monitor:5000/pg_auto_failover \
    --auth scram-sha-256

# On secondary node
pg_autoctl create postgres \
    --pgdata /var/lib/pgsql/data \
    --pgport 5432 \
    --pgctl /usr/bin/pg_ctl \
    --monitor postgres://autoctl@monitor:5000/pg_auto_failover \
    --auth scram-sha-256

# Show cluster state
pg_autoctl show state
```

---

## 6. Multi-Region Deployment

### Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Multi-Region HA                                   │
│                                                                      │
│  Region A (Primary)              Region B (DR)                       │
│  ┌───────────────────┐          ┌───────────────────┐               │
│  │ ┌───────────────┐ │          │ ┌───────────────┐ │               │
│  │ │   Primary     │─┼──Sync───▶│ │ Sync Standby  │ │               │
│  │ └───────────────┘ │          │ └───────────────┘ │               │
│  │        │          │          │        │          │               │
│  │        │          │          │        │          │               │
│  │ ┌───────────────┐ │          │ ┌───────────────┐ │               │
│  │ │ Async Standby │ │          │ │ Async Standby │ │               │
│  │ └───────────────┘ │          │ └───────────────┘ │               │
│  │                   │          │                   │               │
│  │  [Patroni+etcd]   │◄────────▶│  [Patroni+etcd]   │               │
│  └───────────────────┘          └───────────────────┘               │
│                                                                      │
│  Considerations:                                                     │
│  • Sync standby in DR region for zero data loss                     │
│  • Async standbys for read scaling                                  │
│  • Cross-region etcd for consensus                                  │
│  • DNS/Load balancer for region failover                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Synchronous Multi-Region

```yaml
# Patroni config for multi-region sync

bootstrap:
  dcs:
    synchronous_mode: true
    synchronous_node_count: 1
    postgresql:
      parameters:
        synchronous_commit: remote_apply
        synchronous_standby_names: 'region_b_standby'

# Trade-offs:
# • Zero data loss on region failure
# • Higher write latency (cross-region round trip)
# • Consider synchronous_commit = 'remote_write' for lower latency
```

---

## 7. Failover Testing

### Planned Switchover

```bash
# Using Patroni
patronictl switchover postgres-cluster --leader node1 --candidate node2

# Using pg_auto_failover
pg_autoctl perform switchover

# Verify
patronictl list
```

### Chaos Testing

```bash
# Simulate primary failure
sudo systemctl stop postgresql  # On primary

# Watch failover
patronictl list --watch

# Verify application connectivity
psql -h haproxy_vip -p 5000 -c "SELECT pg_is_in_recovery();"
# Should return false (new primary)

# Bring back old primary (becomes replica)
sudo systemctl start postgresql
```

---

## Summary

| Component | Purpose |
|-----------|---------|
| Patroni | Automatic failover, cluster management |
| pg_auto_failover | Simpler alternative to Patroni |
| PgBouncer | Connection pooling |
| HAProxy | Load balancing, health checks |
| etcd/Consul | Distributed consensus |

---

## Further Reading

- Patroni documentation
- PgBouncer documentation
- pg_auto_failover documentation
- HAProxy with PostgreSQL guide
