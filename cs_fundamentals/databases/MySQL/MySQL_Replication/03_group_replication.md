# Group Replication

## Learning Objectives
- Understand Group Replication architecture
- Set up single-primary and multi-primary modes
- Configure for high availability
- Handle node failures and recovery

---

## 1. Group Replication Overview

### What is Group Replication?

Group Replication is a MySQL plugin that enables creating elastic, highly-available, fault-tolerant replication topologies.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Group Replication Architecture                    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      Replication Group                       │    │
│  │                                                              │    │
│  │   ┌──────────┐     ┌──────────┐     ┌──────────┐            │    │
│  │   │  Node 1  │◄───►│  Node 2  │◄───►│  Node 3  │            │    │
│  │   │ (Primary)│     │(Secondary│     │(Secondary│            │    │
│  │   └──────────┘     └──────────┘     └──────────┘            │    │
│  │        │               │                │                    │    │
│  │        └───────────────┴────────────────┘                    │    │
│  │              Group Communication System                      │    │
│  │                   (Paxos-based)                              │    │
│  │                                                              │    │
│  │   Features:                                                  │    │
│  │   • Automatic primary election                               │    │
│  │   • Distributed conflict detection                           │    │
│  │   • Automatic failure detection                              │    │
│  │   • Built-in membership management                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Key Features

| Feature | Description |
|---------|-------------|
| Synchronous replication | All nodes have same data |
| Automatic failover | Primary election on failure |
| Conflict detection | Prevents split-brain |
| Membership service | Dynamic node join/leave |
| Quorum-based decisions | Majority agreement required |

---

## 2. Modes of Operation

### Single-Primary Mode (Default)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Single-Primary Mode                               │
│                                                                      │
│     Writes                         Reads (optional)                  │
│       ↓                                 ↓                            │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐                    │
│   │  Node 1  │────►│  Node 2  │────►│  Node 3  │                    │
│   │ (Primary)│     │(Secondary│     │(Secondary│                    │
│   │   R/W    │     │   R/O    │     │   R/O    │                    │
│   └──────────┘     └──────────┘     └──────────┘                    │
│                                                                      │
│   • Only primary accepts writes                                      │
│   • Secondaries are read-only                                        │
│   • Automatic failover if primary fails                              │
│   • Simplest conflict-free mode                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Multi-Primary Mode

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Multi-Primary Mode                                │
│                                                                      │
│     Writes                Writes                Writes               │
│       ↓                     ↓                     ↓                  │
│   ┌──────────┐     ┌──────────┐     ┌──────────┐                    │
│   │  Node 1  │◄───►│  Node 2  │◄───►│  Node 3  │                    │
│   │ (Primary)│     │ (Primary)│     │ (Primary)│                    │
│   │   R/W    │     │   R/W    │     │   R/W    │                    │
│   └──────────┘     └──────────┘     └──────────┘                    │
│                                                                      │
│   • All nodes accept writes                                          │
│   • Conflicts detected and resolved                                  │
│   • First committer wins                                             │
│   • Requires careful application design                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Prerequisites

### Requirements

```sql
-- 1. InnoDB storage engine only
CREATE TABLE test (...) ENGINE=InnoDB;

-- 2. Primary key on every table
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    ...
);

-- 3. GTID mode enabled
gtid_mode = ON
enforce_gtid_consistency = ON

-- 4. Binary logging with row format
log_bin = binlog
binlog_format = ROW
binlog_checksum = NONE

-- 5. Replication settings
log_slave_updates = ON
master_info_repository = TABLE
relay_log_info_repository = TABLE
transaction_write_set_extraction = XXHASH64
```

### Network Requirements

```
• All nodes must be able to communicate
• Low latency recommended (< 2ms ideal)
• Consistent network bandwidth
• Port 33061 (default) for group communication
```

---

## 4. Single-Primary Setup

### Node 1 Configuration (Bootstrap Node)

```ini
# /etc/mysql/mysql.conf.d/group_replication.cnf

[mysqld]
# Server identity
server_id = 1
bind-address = 0.0.0.0

# GTID mode
gtid_mode = ON
enforce_gtid_consistency = ON

# Binary logging
log_bin = binlog
binlog_format = ROW
binlog_checksum = NONE
log_slave_updates = ON

# Replication repositories
master_info_repository = TABLE
relay_log_info_repository = TABLE

# Transaction write set
transaction_write_set_extraction = XXHASH64

# Group Replication settings
plugin_load_add = 'group_replication.so'
group_replication_group_name = "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
group_replication_start_on_boot = OFF
group_replication_local_address = "node1:33061"
group_replication_group_seeds = "node1:33061,node2:33061,node3:33061"
group_replication_bootstrap_group = OFF

# Single-primary mode (default)
group_replication_single_primary_mode = ON
group_replication_enforce_update_everywhere_checks = OFF
```

### Bootstrap the Group

```sql
-- On node 1 only

-- Set root password plugin
SET SQL_LOG_BIN = 0;
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
GRANT CONNECTION_ADMIN ON *.* TO 'repl'@'%';
GRANT BACKUP_ADMIN ON *.* TO 'repl'@'%';
GRANT GROUP_REPLICATION_STREAM ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN = 1;

-- Configure recovery channel
CHANGE REPLICATION SOURCE TO
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'password'
FOR CHANNEL 'group_replication_recovery';

-- Bootstrap the group (only on first node!)
SET GLOBAL group_replication_bootstrap_group = ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group = OFF;

-- Verify
SELECT * FROM performance_schema.replication_group_members;
```

### Add Additional Nodes

```sql
-- On node 2 and node 3

-- Same user setup
SET SQL_LOG_BIN = 0;
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
GRANT CONNECTION_ADMIN ON *.* TO 'repl'@'%';
GRANT BACKUP_ADMIN ON *.* TO 'repl'@'%';
GRANT GROUP_REPLICATION_STREAM ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN = 1;

-- Configure recovery
CHANGE REPLICATION SOURCE TO
    SOURCE_USER = 'repl',
    SOURCE_PASSWORD = 'password'
FOR CHANNEL 'group_replication_recovery';

-- Join the group (no bootstrap!)
START GROUP_REPLICATION;

-- Verify
SELECT * FROM performance_schema.replication_group_members;
```

---

## 5. Multi-Primary Setup

### Configuration Changes

```ini
# All nodes
[mysqld]
# ... same base config ...

# Multi-primary mode
group_replication_single_primary_mode = OFF
group_replication_enforce_update_everywhere_checks = ON
```

### Enable Multi-Primary

```sql
-- Stop group replication on all nodes
STOP GROUP_REPLICATION;

-- Change mode
SET GLOBAL group_replication_single_primary_mode = OFF;
SET GLOBAL group_replication_enforce_update_everywhere_checks = ON;

-- Restart group (bootstrap first node)
-- Node 1:
SET GLOBAL group_replication_bootstrap_group = ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group = OFF;

-- Other nodes:
START GROUP_REPLICATION;
```

### Multi-Primary Considerations

```sql
-- Avoid conflicts:
-- 1. Use auto-increment with offset
auto_increment_increment = 3
auto_increment_offset = 1  -- Different for each node

-- 2. Or use UUIDs
CREATE TABLE orders (
    id CHAR(36) PRIMARY KEY DEFAULT (UUID()),
    ...
);

-- 3. Avoid overlapping writes to same rows
-- Partition data by node when possible
```

---

## 6. Monitoring Group Replication

### Member Status

```sql
-- View all group members
SELECT * FROM performance_schema.replication_group_members;

+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+
| group_replication_applier | 3E11FA47-71CA-11E1-9E33-C80AA9429562 | node1       | 3306        | ONLINE       | PRIMARY     |
| group_replication_applier | 4E11FA47-71CA-11E1-9E33-C80AA9429563 | node2       | 3306        | ONLINE       | SECONDARY   |
| group_replication_applier | 5E11FA47-71CA-11E1-9E33-C80AA9429564 | node3       | 3306        | ONLINE       | SECONDARY   |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+
```

### Member States

| State | Description |
|-------|-------------|
| ONLINE | Fully functioning member |
| RECOVERING | Catching up with group |
| OFFLINE | Not part of group |
| ERROR | Error state, needs intervention |
| UNREACHABLE | Network partition suspected |

### Transaction Statistics

```sql
-- View transaction stats
SELECT * FROM performance_schema.replication_group_member_stats\G

-- Key fields:
-- COUNT_TRANSACTIONS_IN_QUEUE: Pending transactions
-- COUNT_TRANSACTIONS_CHECKED: Conflict checks
-- COUNT_CONFLICTS_DETECTED: Conflicts (multi-primary)
-- COUNT_TRANSACTIONS_ROWS_VALIDATING: Being certified
```

---

## 7. Failure Handling

### Automatic Failover

```sql
-- When primary fails in single-primary mode:
-- 1. Group detects failure (timeout)
-- 2. Remaining members elect new primary
-- 3. New primary enables writes
-- 4. Automatic (no manual intervention)

-- View failover history
SELECT * FROM performance_schema.replication_group_members;

-- Monitor primary changes
SHOW STATUS LIKE 'group_replication_primary_member';
```

### Recovering Failed Node

```sql
-- If node was cleanly stopped:
START GROUP_REPLICATION;
-- Node will automatically recover

-- If data is inconsistent:
-- 1. Stop MySQL
-- 2. Delete data directory
-- 3. Restart MySQL
-- 4. Clone from existing member:
CLONE INSTANCE FROM 'repl'@'node1':3306 IDENTIFIED BY 'password';
-- 5. After restart:
START GROUP_REPLICATION;
```

### Network Partition Handling

```sql
-- Quorum requirements:
-- Need majority of members for group to function
-- 3 nodes: need 2 (can lose 1)
-- 5 nodes: need 3 (can lose 2)

-- If quorum lost (minority partition):
-- Group becomes read-only
-- Manual intervention required:

-- Force new quorum (use with caution!)
SET GLOBAL group_replication_force_members = 'node1:33061,node2:33061';
```

---

## 8. MySQL InnoDB Cluster

### Complete HA Solution

```
┌─────────────────────────────────────────────────────────────────────┐
│                    InnoDB Cluster Architecture                       │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      MySQL Router                            │    │
│  │         (Connection routing & load balancing)                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ↓                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    Group Replication                         │    │
│  │   ┌────────┐     ┌────────┐     ┌────────┐                  │    │
│  │   │ Node 1 │◄───►│ Node 2 │◄───►│ Node 3 │                  │    │
│  │   └────────┘     └────────┘     └────────┘                  │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              ↓                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    MySQL Shell                               │    │
│  │         (Cluster administration & monitoring)                │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Create Cluster with MySQL Shell

```javascript
// Connect to first node
shell.connect('root@node1:3306')

// Configure instance
dba.configureInstance('root@node1:3306')

// Create cluster
var cluster = dba.createCluster('myCluster')

// Add nodes
cluster.addInstance('root@node2:3306')
cluster.addInstance('root@node3:3306')

// Check status
cluster.status()
```

### MySQL Router Setup

```bash
# Bootstrap router from cluster
mysqlrouter --bootstrap root@node1:3306 --user=mysqlrouter

# Router config creates:
# Port 6446: Read/Write (routes to primary)
# Port 6447: Read-Only (routes to secondaries)

# Application connects to router:
mysql -h router.example.com -P 6446 -u app_user -p
```

---

## Summary

| Mode | Writes | Failover | Use Case |
|------|--------|----------|----------|
| Single-Primary | One node | Automatic | Most applications |
| Multi-Primary | All nodes | N/A | Specific use cases |
| InnoDB Cluster | Via Router | Automatic | Production HA |

---

## Further Reading

- MySQL Group Replication documentation
- InnoDB Cluster documentation
- "Pro MySQL NDB Cluster" book
