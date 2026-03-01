# PostgreSQL Process Architecture

## Learning Objectives
- Understand PostgreSQL's multi-process model
- Learn about the postmaster and backend processes
- Master background worker processes
- Monitor and manage processes effectively

---

## 1. Multi-Process Model

### Overview

Unlike MySQL's multi-threaded architecture, PostgreSQL uses a multi-process model where each client connection is handled by a separate backend process.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Process Model                          │
│                                                                      │
│   Client                     Client                    Client        │
│     │                          │                          │          │
│     │ TCP/IP                   │ TCP/IP                   │ Unix     │
│     │ or Unix Socket           │                          │ Socket   │
│     ▼                          ▼                          ▼          │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                     POSTMASTER                                │   │
│  │                  (postgres main process)                      │   │
│  │                     PID: 1234                                 │   │
│  │                                                               │   │
│  │  • Listens for connections                                    │   │
│  │  • Forks backend processes                                    │   │
│  │  • Manages background workers                                 │   │
│  │  • Handles signals and shutdown                               │   │
│  └──────────────────────────────────────────────────────────────┘   │
│           │                    │                    │                │
│           │ fork()             │ fork()             │ fork()         │
│           ▼                    ▼                    ▼                │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │
│  │   Backend    │    │   Backend    │    │   Backend    │          │
│  │   Process    │    │   Process    │    │   Process    │          │
│  │   PID: 1235  │    │   PID: 1236  │    │   PID: 1237  │          │
│  └──────────────┘    └──────────────┘    └──────────────┘          │
│                                                                      │
│  Each backend:                                                       │
│  • Dedicated to one connection                                       │
│  • Has private memory                                                │
│  • Accesses shared memory                                            │
│  • Terminates when client disconnects                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Benefits of Multi-Process Model

| Benefit | Description |
|---------|-------------|
| Isolation | Crash in one backend doesn't affect others |
| Security | Process-level memory isolation |
| Simplicity | No thread synchronization complexity |
| Stability | More robust than threading |

### Trade-offs

| Aspect | Multi-Process (PostgreSQL) | Multi-Threaded (MySQL) |
|--------|---------------------------|------------------------|
| Memory usage | Higher (per-process) | Lower (shared) |
| Connection overhead | Higher (fork) | Lower (thread create) |
| Context switching | OS-managed | Library-managed |
| Connection pooling | More important | Less critical |

---

## 2. Postmaster Process

### Role and Responsibilities

```sql
-- View postmaster info
SELECT pg_backend_pid();  -- Backend PID
\! ps aux | grep postgres | grep -v grep
```

```bash
# Typical ps output
postgres  1234  ... postgres: postmaster
postgres  1235  ... postgres: checkpointer
postgres  1236  ... postgres: background writer
postgres  1237  ... postgres: walwriter
postgres  1238  ... postgres: autovacuum launcher
postgres  1239  ... postgres: stats collector
postgres  1240  ... postgres: logical replication launcher
postgres  1241  ... postgres: user dbname [local] idle
```

### Postmaster Lifecycle

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Postmaster Startup Sequence                       │
│                                                                      │
│  1. Parse command line and configuration files                       │
│  2. Allocate shared memory                                           │
│  3. Initialize semaphores                                            │
│  4. Start background processes                                       │
│  5. Open listening socket(s)                                         │
│  6. Enter main loop (accept connections)                             │
│                                                                      │
│  For each connection:                                                │
│  1. Accept connection                                                │
│  2. Fork new backend process                                         │
│  3. Backend performs authentication                                  │
│  4. Backend enters command loop                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Backend Processes

### Query Processing Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Backend Query Processing                          │
│                                                                      │
│  Client: SELECT * FROM users WHERE id = 1;                           │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 1. PARSER                                                    │    │
│  │    • Lexical analysis                                        │    │
│  │    • Syntax checking                                         │    │
│  │    • Create parse tree                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 2. ANALYZER/REWRITER                                         │    │
│  │    • Semantic analysis                                       │    │
│  │    • View expansion                                          │    │
│  │    • Rule application                                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 3. PLANNER/OPTIMIZER                                         │    │
│  │    • Generate execution plans                                │    │
│  │    • Cost estimation                                         │    │
│  │    • Select optimal plan                                     │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ 4. EXECUTOR                                                  │    │
│  │    • Execute plan nodes                                      │    │
│  │    • Access storage via buffer manager                       │    │
│  │    • Return results to client                                │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Backend Memory

```sql
-- Each backend has private memory for:
-- • Sort operations (work_mem)
-- • Hash operations
-- • Query plans (plan cache)
-- • Temporary tables

-- View work_mem setting
SHOW work_mem;

-- Per-session setting
SET work_mem = '256MB';

-- View backend memory usage
SELECT pid, usename, backend_type,
       pg_size_pretty(backend_memory_contexts.total_bytes) as memory
FROM pg_stat_activity
JOIN LATERAL (
    SELECT SUM(total_bytes) as total_bytes
    FROM pg_backend_memory_contexts
) backend_memory_contexts ON true
WHERE pid = pg_backend_pid();
```

---

## 4. Background Processes

### Essential Background Workers

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Background Processes                              │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ CHECKPOINTER                                                 │    │
│  │ • Writes dirty buffers to disk periodically                 │    │
│  │ • Creates checkpoint records in WAL                         │    │
│  │ • Enables crash recovery from known point                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ BACKGROUND WRITER (bgwriter)                                 │    │
│  │ • Gradually writes dirty buffers                            │    │
│  │ • Reduces checkpoint I/O spikes                             │    │
│  │ • Keeps clean pages available                               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ WAL WRITER                                                   │    │
│  │ • Flushes WAL buffers to disk                               │    │
│  │ • Ensures durability                                        │    │
│  │ • Wakes on commit or timeout                                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ AUTOVACUUM LAUNCHER                                          │    │
│  │ • Starts autovacuum workers                                 │    │
│  │ • Based on table statistics                                 │    │
│  │ • Reclaims dead tuples                                      │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ STATS COLLECTOR                                              │    │
│  │ • Collects runtime statistics                               │    │
│  │ • Writes to pg_stat_* views                                 │    │
│  │ • Replaced in PostgreSQL 15+ by shared memory stats        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ LOGICAL REPLICATION LAUNCHER                                 │    │
│  │ • Manages logical replication workers                       │    │
│  │ • Starts apply workers for subscriptions                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Viewing Processes

```sql
-- View all PostgreSQL processes
SELECT pid, usename, application_name,
       backend_type, state, query
FROM pg_stat_activity;

-- Background workers only
SELECT pid, backend_type
FROM pg_stat_activity
WHERE backend_type != 'client backend';

-- System view of processes
\! ps aux | grep postgres
```

---

## 5. Connection Handling

### Connection Limits

```sql
-- View connection settings
SHOW max_connections;
SHOW superuser_reserved_connections;

-- Current connections
SELECT count(*) FROM pg_stat_activity;

-- Connections by database
SELECT datname, count(*)
FROM pg_stat_activity
GROUP BY datname;

-- Connections by user
SELECT usename, count(*)
FROM pg_stat_activity
GROUP BY usename;

-- Connections by state
SELECT state, count(*)
FROM pg_stat_activity
GROUP BY state;
```

### Connection Configuration

```ini
# postgresql.conf

# Maximum connections
max_connections = 200

# Reserved for superuser
superuser_reserved_connections = 3

# Authentication timeout
authentication_timeout = 60s

# Connection timeout (0 = disabled)
tcp_keepalives_idle = 60
tcp_keepalives_interval = 10
tcp_keepalives_count = 10
```

### Connection Pooling (Essential for PostgreSQL)

```
┌─────────────────────────────────────────────────────────────────────┐
│              Connection Pooling with PgBouncer                       │
│                                                                      │
│   Application Servers                                                │
│   ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                           │
│   │ App │ │ App │ │ App │ │ App │ │ App │  (100s of connections)    │
│   └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘                           │
│      └──────┬───────┴───────┬───────┘                               │
│             ▼               ▼                                        │
│   ┌─────────────────────────────────────────────────────────┐       │
│   │                    PgBouncer                             │       │
│   │              (Connection Pool)                           │       │
│   │         Maintains pool of connections                    │       │
│   └─────────────────────────────────────────────────────────┘       │
│                  │               │               │                   │
│                  ▼               ▼               ▼                   │
│            ┌──────────┐   ┌──────────┐   ┌──────────┐               │
│            │ Backend  │   │ Backend  │   │ Backend  │  (20-50       │
│            │ Process  │   │ Process  │   │ Process  │   connections)│
│            └──────────┘   └──────────┘   └──────────┘               │
│                                                                      │
│   Pool Modes:                                                        │
│   • Session: Connection per client session                           │
│   • Transaction: Connection per transaction                          │
│   • Statement: Connection per statement (limited)                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Process Management

### Starting and Stopping

```bash
# Using pg_ctl
pg_ctl start -D /var/lib/postgresql/data
pg_ctl stop -D /var/lib/postgresql/data -m smart   # Wait for clients
pg_ctl stop -D /var/lib/postgresql/data -m fast    # Rollback active transactions
pg_ctl stop -D /var/lib/postgresql/data -m immediate  # Abort (crash-like)
pg_ctl restart -D /var/lib/postgresql/data
pg_ctl reload -D /var/lib/postgresql/data  # Reload configuration

# Using systemd
systemctl start postgresql
systemctl stop postgresql
systemctl restart postgresql
systemctl reload postgresql
```

### Signaling Backends

```sql
-- Cancel a query (SIGINT)
SELECT pg_cancel_backend(pid);

-- Terminate a connection (SIGTERM)
SELECT pg_terminate_backend(pid);

-- Example: Kill long-running queries
SELECT pg_cancel_backend(pid)
FROM pg_stat_activity
WHERE state = 'active'
  AND query_start < NOW() - INTERVAL '5 minutes'
  AND pid != pg_backend_pid();
```

### Process Signals

```bash
# Signals handled by postmaster:
# SIGHUP  - Reload configuration
# SIGINT  - Smart shutdown
# SIGQUIT - Fast shutdown
# SIGTERM - Fast shutdown
# SIGUSR1 - Used internally
# SIGUSR2 - Used internally

# Reload configuration
kill -HUP `cat /var/lib/postgresql/data/postmaster.pid | head -1`
# Or
SELECT pg_reload_conf();
```

---

## 7. Monitoring Processes

### pg_stat_activity

```sql
-- Detailed process view
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    backend_start,
    xact_start,
    query_start,
    state,
    wait_event_type,
    wait_event,
    LEFT(query, 50) as query_preview
FROM pg_stat_activity
WHERE backend_type = 'client backend'
ORDER BY query_start DESC NULLS LAST;

-- Find blocking processes
SELECT
    blocked.pid AS blocked_pid,
    blocked.query AS blocked_query,
    blocking.pid AS blocking_pid,
    blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
    ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE blocked.pid != blocked.pid;
```

### Wait Events

```sql
-- Current wait events
SELECT wait_event_type, wait_event, count(*)
FROM pg_stat_activity
WHERE wait_event IS NOT NULL
GROUP BY wait_event_type, wait_event
ORDER BY count(*) DESC;

-- Wait event types:
-- LWLock: Lightweight lock
-- Lock: Heavy lock
-- BufferPin: Buffer pin wait
-- Activity: Background worker activity
-- Extension: Extension wait
-- Client: Waiting for client
-- IPC: Inter-process communication
-- Timeout: Timeout wait
-- IO: I/O wait
```

---

## Summary

| Process | Role |
|---------|------|
| Postmaster | Main daemon, forks backends |
| Backend | Handles client connections |
| Checkpointer | Periodic checkpoint writes |
| Background Writer | Gradual dirty buffer writes |
| WAL Writer | Flushes WAL to disk |
| Autovacuum | Automatic maintenance |
| Stats Collector | Gathers statistics |

---

## Further Reading

- PostgreSQL Server Programming documentation
- "PostgreSQL Internals" by Hironobu Suzuki
- PostgreSQL Process Architecture wiki
