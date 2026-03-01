# MySQL Server Architecture

## Learning Objectives
- Understand MySQL's client-server architecture
- Learn the query execution flow
- Master connection handling and threading models
- Understand the SQL layer components

---

## 1. Client-Server Model

MySQL operates on a client-server architecture where the server process (`mysqld`) handles all database operations.

```
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│  mysql CLI     │  │  Application   │  │  MySQL         │
│  Client        │  │  (Python, PHP) │  │  Workbench     │
└───────┬────────┘  └───────┬────────┘  └───────┬────────┘
        │                   │                   │
        │    MySQL Protocol (TCP/IP or Socket)  │
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                   ┌────────────────────┐
                   │    mysqld          │
                   │  (MySQL Server)    │
                   └────────────────────┘
```

### Connection Methods

```bash
# TCP/IP Connection (default port 3306)
mysql -h 192.168.1.100 -P 3306 -u root -p

# Unix Socket (local connections on Linux/macOS)
mysql -S /var/run/mysqld/mysqld.sock -u root -p

# Named Pipe (Windows)
mysql --pipe --user=root -p

# Shared Memory (Windows, same host)
mysql --protocol=memory --shared-memory-base-name=MySQL -u root -p
```

---

## 2. Connection Handling

### Thread-Per-Connection Model (Traditional)

```
┌─────────────────────────────────────────────────────────┐
│                    mysqld Process                        │
│  ┌──────────────────────────────────────────────────┐   │
│  │               Connection Manager                  │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │   │
│  │  │ Thread  │ │ Thread  │ │ Thread  │ │ Thread  │ │   │
│  │  │ (conn1) │ │ (conn2) │ │ (conn3) │ │ (conn4) │ │   │
│  │  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ │   │
│  └───────┼──────────┼──────────┼──────────┼────────┘   │
│          │          │          │          │             │
│          ▼          ▼          ▼          ▼             │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Shared Resources                     │   │
│  │    (Buffer Pool, Query Cache, Log Buffers)        │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Thread Pool (Enterprise/MariaDB)

```
┌─────────────────────────────────────────────────────────┐
│                    Thread Pool                           │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Thread Group 1                      │    │
│  │  ┌────────┐ ┌────────┐ ← Handles multiple       │    │
│  │  │Worker 1│ │Worker 2│   connections            │    │
│  │  └────────┘ └────────┘                          │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Thread Group 2                      │    │
│  │  ┌────────┐ ┌────────┐                          │    │
│  │  │Worker 3│ │Worker 4│                          │    │
│  │  └────────┘ └────────┘                          │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### Connection Configuration

```sql
-- View current connections
SHOW PROCESSLIST;
SHOW STATUS LIKE 'Threads%';

-- Connection limits
SHOW VARIABLES LIKE 'max_connections';           -- Default: 151
SHOW VARIABLES LIKE 'max_user_connections';      -- Per-user limit
SHOW VARIABLES LIKE 'wait_timeout';              -- Idle timeout
SHOW VARIABLES LIKE 'interactive_timeout';       -- Interactive session timeout

-- Set connection limits
SET GLOBAL max_connections = 500;
SET GLOBAL wait_timeout = 28800;  -- 8 hours
```

---

## 3. Query Execution Flow

### Complete Query Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│ Client: SELECT * FROM users WHERE status = 'active' ORDER BY id │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 1. CONNECTION HANDLING                                            │
│    - Authentication (mysql.user table)                            │
│    - Authorization check                                          │
│    - Thread assignment                                            │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 2. QUERY CACHE (Removed in MySQL 8.0)                             │
│    - Check if exact query result is cached                        │
│    - If hit, return cached result immediately                     │
└──────────────────────────────┬───────────────────────────────────┘
                               │ (cache miss)
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 3. PARSER                                                         │
│    - Lexical analysis (tokenization)                              │
│    - Syntax validation                                            │
│    - Create parse tree                                            │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 4. PREPROCESSOR                                                   │
│    - Resolve table and column names                               │
│    - Check privileges                                             │
│    - Verify semantic correctness                                  │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 5. OPTIMIZER                                                      │
│    - Analyze query                                                │
│    - Generate execution plans                                     │
│    - Estimate costs                                               │
│    - Select optimal plan                                          │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 6. EXECUTOR                                                       │
│    - Execute plan via Storage Engine API                          │
│    - Fetch data from storage engine                               │
│    - Apply WHERE filters, ORDER BY, LIMIT                         │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 7. STORAGE ENGINE (InnoDB)                                        │
│    - Buffer pool lookup                                           │
│    - Disk I/O if needed                                           │
│    - Return rows to executor                                      │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│ 8. RESULT SET                                                     │
│    - Format results                                               │
│    - Send to client                                               │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. SQL Layer Components

### 4.1 Parser

Converts SQL text into an internal structure.

```sql
-- Parser validates syntax
SELECT * FORM users;  -- ERROR 1064: Syntax error near 'FORM'

-- Parser creates parse tree
SELECT id, name FROM users WHERE active = 1;

/*
Parse Tree:
    SELECT_STMT
    ├── SELECT_LIST
    │   ├── COLUMN: id
    │   └── COLUMN: name
    ├── FROM_CLAUSE
    │   └── TABLE: users
    └── WHERE_CLAUSE
        └── CONDITION: active = 1
*/
```

### 4.2 Preprocessor

Validates semantics and checks permissions.

```sql
-- Preprocessor validates object existence
SELECT * FROM nonexistent_table;  -- ERROR 1146: Table doesn't exist

-- Preprocessor resolves aliases
SELECT u.id, u.name
FROM users u
WHERE u.active = 1;  -- 'u' resolved to 'users'

-- Preprocessor checks column ambiguity
SELECT id FROM users, orders;  -- ERROR if both tables have 'id'
```

### 4.3 Optimizer

Generates and evaluates execution plans.

```sql
-- View optimizer's chosen plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Optimizer statistics
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE status = 'active';

-- Optimizer hints (MySQL 8.0+)
SELECT /*+ INDEX(users idx_status) */ * FROM users WHERE status = 'active';
SELECT /*+ NO_INDEX(users idx_status) */ * FROM users WHERE status = 'active';
SELECT /*+ JOIN_ORDER(orders, users) */ * FROM orders JOIN users ON orders.user_id = users.id;
```

### 4.4 Executor

Executes the optimized plan.

```sql
-- Executor operations visible via performance_schema
SELECT * FROM performance_schema.events_statements_current\G

-- Handler operations (storage engine calls)
SHOW STATUS LIKE 'Handler%';
/*
Handler_read_first      - Index scan started
Handler_read_key        - Row read via index
Handler_read_next       - Next row in index order
Handler_read_rnd        - Random row read
Handler_write           - Row inserted
*/
```

---

## 5. Server Threads

### Background Threads

```
┌─────────────────────────────────────────────────────────────────┐
│                    MySQL Background Threads                      │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  Master Thread  │  │  IO Threads     │  │  Purge Thread   │  │
│  │  - Checkpoints  │  │  - Read IO      │  │  - Undo cleanup │  │
│  │  - Flushing     │  │  - Write IO     │  │  - History list │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │ Page Cleaner    │  │ Log Writer      │  │ Replication     │  │
│  │ - Dirty page    │  │ - Redo log      │  │ - Binlog dump   │  │
│  │   flushing      │  │   writes        │  │ - Slave IO/SQL  │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

```sql
-- View InnoDB thread configuration
SHOW VARIABLES LIKE 'innodb_read_io_threads';   -- Default: 4
SHOW VARIABLES LIKE 'innodb_write_io_threads';  -- Default: 4
SHOW VARIABLES LIKE 'innodb_purge_threads';     -- Default: 4
SHOW VARIABLES LIKE 'innodb_page_cleaners';     -- Default: 4

-- View running threads
SELECT * FROM performance_schema.threads WHERE TYPE = 'BACKGROUND';
```

---

## 6. Server Files

### Data Directory Structure

```
/var/lib/mysql/                    # Data directory
├── mysql/                         # System database
│   ├── user.ibd                   # User accounts
│   ├── db.ibd                     # Database privileges
│   └── ...
├── performance_schema/            # Performance monitoring
├── sys/                           # sys schema (views)
├── mydb/                          # User database
│   ├── users.ibd                  # InnoDB table
│   ├── orders.ibd                 # InnoDB table
│   └── ...
├── ib_logfile0                    # Redo log file 1
├── ib_logfile1                    # Redo log file 2
├── ibdata1                        # System tablespace
├── undo_001                       # Undo tablespace
├── undo_002                       # Undo tablespace
├── mysql.ibd                      # Data dictionary (8.0)
├── binlog.000001                  # Binary log
├── binlog.index                   # Binary log index
├── slow_query.log                 # Slow query log
├── error.log                      # Error log
└── mysql.sock                     # Unix socket
```

### Configuration File Locations

```bash
# Linux
/etc/my.cnf
/etc/mysql/my.cnf
~/.my.cnf

# macOS (Homebrew)
/usr/local/etc/my.cnf
/opt/homebrew/etc/my.cnf

# Windows
C:\ProgramData\MySQL\MySQL Server 8.0\my.ini
```

---

## 7. Server Status and Monitoring

### Key Status Variables

```sql
-- Connection statistics
SHOW GLOBAL STATUS LIKE 'Connections';           -- Total connection attempts
SHOW GLOBAL STATUS LIKE 'Threads_connected';     -- Currently connected
SHOW GLOBAL STATUS LIKE 'Threads_running';       -- Currently executing
SHOW GLOBAL STATUS LIKE 'Max_used_connections';  -- Peak connections

-- Query statistics
SHOW GLOBAL STATUS LIKE 'Questions';             -- Total queries
SHOW GLOBAL STATUS LIKE 'Queries';               -- Including stored routines
SHOW GLOBAL STATUS LIKE 'Slow_queries';          -- Slow query count
SHOW GLOBAL STATUS LIKE 'Com_select';            -- SELECT count
SHOW GLOBAL STATUS LIKE 'Com_insert';            -- INSERT count

-- InnoDB statistics
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read_requests';  -- Logical reads
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_reads';          -- Physical reads
SHOW GLOBAL STATUS LIKE 'Innodb_rows_read';
SHOW GLOBAL STATUS LIKE 'Innodb_rows_inserted';
```

### Performance Schema Queries

```sql
-- Top queries by execution time
SELECT DIGEST_TEXT, COUNT_STAR, AVG_TIMER_WAIT/1000000000 AS avg_ms
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;

-- Current running queries
SELECT * FROM performance_schema.events_statements_current
WHERE SQL_TEXT IS NOT NULL\G

-- Connection statistics by user
SELECT USER, CURRENT_CONNECTIONS, TOTAL_CONNECTIONS
FROM performance_schema.accounts
WHERE USER IS NOT NULL;
```

---

## 8. Server Configuration Best Practices

### Essential my.cnf Settings

```ini
[mysqld]
# Connection Settings
max_connections = 500
max_connect_errors = 1000000
wait_timeout = 28800
interactive_timeout = 28800

# Thread Settings
thread_cache_size = 100
thread_stack = 256K

# Query Execution
max_allowed_packet = 64M
tmp_table_size = 64M
max_heap_table_size = 64M
sort_buffer_size = 2M
join_buffer_size = 2M

# Logging
log_error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
log_queries_not_using_indexes = 1

# Binary Logging (for replication)
log_bin = /var/log/mysql/binlog
binlog_format = ROW
binlog_expire_logs_seconds = 604800  # 7 days
sync_binlog = 1

# Character Set
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci

# Security
local_infile = 0
skip_symbolic_links = 1
```

---

## Summary

| Component | Role |
|-----------|------|
| Connection Layer | Authentication, thread management |
| Parser | SQL syntax validation, parse tree |
| Preprocessor | Semantic validation, privilege check |
| Optimizer | Execution plan generation and selection |
| Executor | Plan execution via storage engine API |
| Storage Engine | Data storage, retrieval, transactions |

---

## Further Reading

- MySQL Server Administration documentation
- Performance Schema reference
- MySQL Internals manual
