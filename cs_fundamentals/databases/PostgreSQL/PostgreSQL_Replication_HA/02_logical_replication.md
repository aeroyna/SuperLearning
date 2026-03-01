# Logical Replication

## Learning Objectives
- Understand logical replication concepts
- Configure publishers and subscribers
- Handle schema changes and conflicts
- Use logical replication for migrations

---

## 1. Logical Replication Overview

### How It Differs from Streaming

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Logical vs Streaming Replication                  │
│                                                                      │
│  STREAMING (Physical):                                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Primary                   Standby                          │    │
│  │  ┌──────────┐             ┌──────────┐                      │    │
│  │  │ Block    │────WAL────▶│ Block    │                      │    │
│  │  │ Changes  │   bytes    │ Replay   │                      │    │
│  │  └──────────┘             └──────────┘                      │    │
│  │  • Identical bytes                                          │    │
│  │  • Same PostgreSQL version                                  │    │
│  │  • Entire cluster replicated                                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  LOGICAL:                                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Publisher                Subscriber                        │    │
│  │  ┌──────────┐             ┌──────────┐                      │    │
│  │  │ INSERT   │───Logical──▶│ INSERT   │                      │    │
│  │  │ UPDATE   │   Changes  │ UPDATE   │                      │    │
│  │  │ DELETE   │  (decoded) │ DELETE   │                      │    │
│  │  └──────────┘             └──────────┘                      │    │
│  │  • Row-level changes                                        │    │
│  │  • Cross-version compatible                                 │    │
│  │  • Selective tables/columns                                 │    │
│  │  • Subscriber can have additional objects                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Use Cases

```
When to use Logical Replication:
• Upgrade PostgreSQL with minimal downtime
• Replicate subset of tables to analytics DB
• Consolidate multiple databases into one
• Replicate between different architectures
• Bi-directional replication (with conflict handling)
• Data integration between systems

When NOT to use:
• Need full cluster copy (use streaming)
• Replicating DDL automatically
• Very high write throughput (performance overhead)
• Need failover capability (no automatic promotion)
```

---

## 2. Publisher Configuration

### Enable Logical Replication

```ini
# postgresql.conf on PUBLISHER

# Must be 'logical' for logical replication
wal_level = logical

# Slots for subscribers
max_replication_slots = 10

# Sender processes
max_wal_senders = 10
```

### Create Publication

```sql
-- Connect to publisher database

-- Publish specific tables
CREATE PUBLICATION my_publication
FOR TABLE users, orders, products;

-- Publish all tables in schema
CREATE PUBLICATION schema_pub
FOR TABLES IN SCHEMA public;

-- Publish all tables
CREATE PUBLICATION all_tables
FOR ALL TABLES;

-- Publish with row filter (PostgreSQL 15+)
CREATE PUBLICATION active_users_pub
FOR TABLE users WHERE (status = 'active');

-- Publish specific columns (PostgreSQL 15+)
CREATE PUBLICATION partial_pub
FOR TABLE users (id, email, created_at);

-- With specific operations
CREATE PUBLICATION inserts_only
FOR TABLE logs
WITH (publish = 'insert');  -- insert, update, delete, truncate
```

### Managing Publications

```sql
-- View publications
SELECT * FROM pg_publication;

-- View publication tables
SELECT * FROM pg_publication_tables;

-- Add table to publication
ALTER PUBLICATION my_publication ADD TABLE new_table;

-- Remove table
ALTER PUBLICATION my_publication DROP TABLE old_table;

-- Change published operations
ALTER PUBLICATION my_publication
SET (publish = 'insert, update');

-- Drop publication
DROP PUBLICATION my_publication;
```

---

## 3. Subscriber Configuration

### Create Subscription

```sql
-- Connect to subscriber database

-- Create matching tables first (schema not replicated!)
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255),
    status VARCHAR(20)
);

-- Create subscription
CREATE SUBSCRIPTION my_subscription
CONNECTION 'host=publisher.example.com port=5432 dbname=mydb user=replicator password=secret'
PUBLICATION my_publication;

-- Options
CREATE SUBSCRIPTION my_subscription
CONNECTION 'host=publisher port=5432 dbname=mydb user=replicator'
PUBLICATION my_publication
WITH (
    copy_data = true,      -- Initial table sync (default true)
    enabled = true,        -- Start immediately (default true)
    create_slot = true,    -- Create replication slot (default true)
    slot_name = 'my_slot', -- Custom slot name
    synchronous_commit = off  -- Faster but less durable
);
```

### Managing Subscriptions

```sql
-- View subscriptions
SELECT * FROM pg_subscription;

-- View subscription status
SELECT * FROM pg_stat_subscription;

-- Disable subscription (pause)
ALTER SUBSCRIPTION my_subscription DISABLE;

-- Enable subscription (resume)
ALTER SUBSCRIPTION my_subscription ENABLE;

-- Refresh publication (after adding tables)
ALTER SUBSCRIPTION my_subscription REFRESH PUBLICATION;

-- Change connection
ALTER SUBSCRIPTION my_subscription
CONNECTION 'host=new_publisher port=5432 ...';

-- Drop subscription
DROP SUBSCRIPTION my_subscription;
```

---

## 4. Replica Identity

### Why Replica Identity Matters

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Replica Identity                                  │
│                                                                      │
│  For UPDATE/DELETE, subscriber needs to identify which row to modify│
│                                                                      │
│  Options:                                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  default:  Use primary key (if exists)                      │    │
│  │  using index: Use specified unique index                    │    │
│  │  full: Log all columns (allows update without PK)           │    │
│  │  nothing: No identity (only INSERT works)                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Table without primary key and REPLICA IDENTITY DEFAULT:            │
│  → UPDATE/DELETE won't replicate (ERROR)                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Setting Replica Identity

```sql
-- Use primary key (default)
ALTER TABLE users REPLICA IDENTITY DEFAULT;

-- Use specific unique index
CREATE UNIQUE INDEX users_email_idx ON users (email);
ALTER TABLE users REPLICA IDENTITY USING INDEX users_email_idx;

-- Log all columns (for tables without unique constraint)
ALTER TABLE logs REPLICA IDENTITY FULL;

-- No identity (INSERT only)
ALTER TABLE metrics REPLICA IDENTITY NOTHING;

-- Check current setting
SELECT relname, relreplident
FROM pg_class
WHERE relname = 'users';
-- d = default, i = index, f = full, n = nothing
```

---

## 5. Initial Synchronization

### Copy Data Phase

```sql
-- By default, subscription copies existing data
CREATE SUBSCRIPTION my_sub
CONNECTION '...'
PUBLICATION my_pub
WITH (copy_data = true);  -- Default

-- Skip initial copy (table already has data)
CREATE SUBSCRIPTION my_sub
CONNECTION '...'
PUBLICATION my_pub
WITH (copy_data = false);

-- Monitor sync progress
SELECT
    srsubid,
    srrelid::regclass AS table_name,
    srsubstate,  -- i = init, d = data copying, s = synced, r = ready
    srsublsn
FROM pg_subscription_rel;
```

### Large Table Sync

```sql
-- For large tables, initial sync can take time
-- Monitor progress:
SELECT
    relid::regclass AS table_name,
    phase,
    heap_blks_total,
    heap_blks_scanned,
    ROUND(100.0 * heap_blks_scanned / NULLIF(heap_blks_total, 0), 2) AS pct_done
FROM pg_stat_progress_copy
WHERE command = 'COPY FROM';
```

---

## 6. Handling Conflicts

### Conflict Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Logical Replication Conflicts                     │
│                                                                      │
│  INSERT conflict:                                                    │
│  • Row with same primary key exists on subscriber                   │
│  • Result: Replication stops with error                             │
│                                                                      │
│  UPDATE/DELETE conflict:                                             │
│  • Row doesn't exist on subscriber                                   │
│  • Logged as warning, replication continues                         │
│                                                                      │
│  Solutions:                                                          │
│  1. Fix data and restart replication                                │
│  2. Skip the transaction                                            │
│  3. Use triggers for custom conflict resolution                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Resolving Conflicts

```sql
-- Check subscription status
SELECT * FROM pg_stat_subscription;

-- View error in logs
-- Look for: ERROR: duplicate key value violates unique constraint

-- Option 1: Fix subscriber data
DELETE FROM users WHERE id = 123;  -- Remove conflicting row
-- Replication will retry

-- Option 2: Skip the transaction on subscriber
-- First find the LSN to skip to
SELECT * FROM pg_replication_origin_status;

-- Advance past the problem
ALTER SUBSCRIPTION my_subscription DISABLE;
SELECT pg_replication_origin_advance('pg_<subid>', '<next_lsn>');
ALTER SUBSCRIPTION my_subscription ENABLE;
```

### Preventing Conflicts

```sql
-- Use ON CONFLICT for insertable subscriber tables
-- (Only if subscriber also receives writes)

-- Consider partition by source to avoid conflicts
-- Publisher A: users with id % 2 = 0
-- Publisher B: users with id % 2 = 1
```

---

## 7. Schema Changes

### DDL Not Replicated

```sql
-- Publisher adds column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
INSERT INTO users (id, email, phone) VALUES (1, 'a@b.com', '123');

-- Subscriber doesn't have column!
-- Result: Error or column ignored depending on version

-- Solution: Apply DDL on both sides
-- Subscriber:
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
-- Then refresh:
ALTER SUBSCRIPTION my_sub REFRESH PUBLICATION;
```

### Schema Change Workflow

```sql
-- 1. Pause subscription (optional but safer)
ALTER SUBSCRIPTION my_sub DISABLE;

-- 2. Apply DDL on subscriber
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- 3. Apply DDL on publisher
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- 4. Resume subscription
ALTER SUBSCRIPTION my_sub ENABLE;

-- For column drops/renames, reverse order
-- (Apply on subscriber last)
```

---

## 8. Monitoring Logical Replication

### Publisher Monitoring

```sql
-- Replication slots
SELECT
    slot_name,
    plugin,
    slot_type,
    active,
    restart_lsn,
    confirmed_flush_lsn,
    pg_wal_lsn_diff(pg_current_wal_lsn(), confirmed_flush_lsn) AS lag_bytes
FROM pg_replication_slots
WHERE slot_type = 'logical';

-- WAL senders for logical replication
SELECT
    pid,
    application_name,
    client_addr,
    state,
    sent_lsn,
    write_lsn
FROM pg_stat_replication
WHERE state = 'streaming';
```

### Subscriber Monitoring

```sql
-- Subscription status
SELECT
    subname,
    pid,
    relid::regclass AS table_name,
    received_lsn,
    last_msg_send_time,
    last_msg_receipt_time,
    latest_end_lsn
FROM pg_stat_subscription;

-- Replication lag (requires tracking origin)
SELECT
    pg_wal_lsn_diff(pg_current_wal_lsn(), remote_lsn) AS lag_bytes
FROM pg_replication_origin_status;
```

---

## 9. Use Cases

### Zero-Downtime Upgrade

```sql
-- Upgrade PostgreSQL 14 → 16

-- 1. Set up new PG16 instance
-- 2. Create subscription to PG14 (logical replication)
CREATE SUBSCRIPTION upgrade_sub
CONNECTION 'host=pg14 dbname=mydb user=replicator'
PUBLICATION all_tables;

-- 3. Wait for sync
SELECT * FROM pg_subscription_rel WHERE srsubstate != 'r';

-- 4. Stop writes to old system
-- 5. Wait for final sync
-- 6. Point application to new system
-- 7. Drop subscription
DROP SUBSCRIPTION upgrade_sub;
```

### Data Consolidation

```sql
-- Consolidate multiple sources into one

-- Source 1
CREATE PUBLICATION source1_pub FOR TABLE users, orders;

-- Source 2
CREATE PUBLICATION source2_pub FOR TABLE users, orders;

-- Consolidated subscriber
CREATE SUBSCRIPTION source1_sub
CONNECTION 'host=source1 ...'
PUBLICATION source1_pub;

CREATE SUBSCRIPTION source2_sub
CONNECTION 'host=source2 ...'
PUBLICATION source2_pub;

-- Handle conflicts with different ID ranges or transforms
```

---

## 10. Limitations

### What's NOT Replicated

```
Logical Replication Limitations:
• DDL (CREATE, ALTER, DROP) - manual sync required
• Sequences - not replicated, must sync separately
• Large objects
• Truncate (optional, disabled by default)
• Foreign tables
• Materialized view data
• Partition root tables (use partitions directly)

Other considerations:
• Subscriber can have indexes, triggers (may affect performance)
• Foreign key checks on subscriber can cause conflicts
• No automatic failover (unlike streaming replication)
```

### Workarounds

```sql
-- Replicate sequence values periodically
-- On subscriber, set sequence:
SELECT setval('users_id_seq', (SELECT MAX(id) FROM users) + 1000);

-- Or use UUIDs instead of sequences

-- For TRUNCATE, enable if needed:
CREATE PUBLICATION with_truncate
FOR TABLE users
WITH (publish = 'insert, update, delete, truncate');
```

---

## Summary

| Feature | Description |
|---------|-------------|
| Publication | Defines what data to replicate |
| Subscription | Connects to publication, receives changes |
| Replica Identity | How to identify rows for UPDATE/DELETE |
| Conflict Handling | Manual resolution required |
| Schema Changes | DDL not replicated, manual sync |

---

## Further Reading

- PostgreSQL Logical Replication documentation
- Logical Decoding
- pg_recvlogical command
