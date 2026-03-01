# Buffer Pool and Caching

## 1. Introduction

The **buffer pool** is an in-memory cache that stores frequently accessed data pages. It's one of the most critical components for database performance, as disk I/O is orders of magnitude slower than memory access.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BUFFER POOL OVERVIEW                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                        APPLICATION                               │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                              │                                              │
│                              ▼                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                     BUFFER POOL (RAM)                            │      │
│   │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐        │      │
│   │  │ Page 1 │ │ Page 5 │ │ Page 2 │ │ Page 8 │ │ Page 3 │  ...   │      │
│   │  └────────┘ └────────┘ └────────┘ └────────┘ └────────┘        │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                              │                                              │
│                              │ Cache Miss                                   │
│                              ▼                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                        DISK (Storage)                            │      │
│   │  [Page 1][Page 2][Page 3][Page 4][Page 5][Page 6][Page 7]...    │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                                                                              │
│   Speed comparison:                                                         │
│   • RAM: ~100 nanoseconds                                                  │
│   • SSD: ~100 microseconds (1000x slower)                                  │
│   • HDD: ~10 milliseconds (100,000x slower)                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Buffer Pool Architecture

### 2.1 Page Management

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BUFFER POOL COMPONENTS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                      BUFFER POOL                                 │      │
│   │                                                                   │      │
│   │   ┌─────────────────────────────────────────────────────────┐   │      │
│   │   │              Page Hash Table                             │   │      │
│   │   │   { (tablespace, page_id) → buffer_frame_ptr }          │   │      │
│   │   └─────────────────────────────────────────────────────────┘   │      │
│   │                                                                   │      │
│   │   ┌─────────────────────────────────────────────────────────┐   │      │
│   │   │              Buffer Frames (Fixed Size)                  │   │      │
│   │   │  ┌────────┬────────┬────────┬────────┬────────┐        │   │      │
│   │   │  │Frame 0 │Frame 1 │Frame 2 │Frame 3 │  ...   │        │   │      │
│   │   │  │[Page X]│[Page Y]│[Free]  │[Page Z]│        │        │   │      │
│   │   │  │dirty=1 │dirty=0 │        │dirty=1 │        │        │   │      │
│   │   │  │pin=2   │pin=0   │        │pin=1   │        │        │   │      │
│   │   │  └────────┴────────┴────────┴────────┴────────┘        │   │      │
│   │   └─────────────────────────────────────────────────────────┘   │      │
│   │                                                                   │      │
│   │   ┌─────────────────────────────────────────────────────────┐   │      │
│   │   │              Free List / LRU List                        │   │      │
│   │   │   Frame 2 → Frame 7 → Frame 12 → ...                    │   │      │
│   │   └─────────────────────────────────────────────────────────┘   │      │
│   │                                                                   │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                                                                              │
│   Each buffer frame contains:                                               │
│   • Page data (actual content)                                             │
│   • Dirty flag (modified since read?)                                      │
│   • Pin count (number of users)                                            │
│   • Reference bit (recently accessed?)                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Page Lifecycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PAGE LIFECYCLE                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. PAGE REQUEST                                                           │
│      Query needs page X                                                     │
│                 │                                                           │
│                 ▼                                                           │
│   2. HASH TABLE LOOKUP                                                      │
│      Is page X in buffer pool?                                             │
│           │                                                                 │
│      ┌────┴────┐                                                           │
│      YES      NO                                                           │
│      │         │                                                           │
│      │         ▼                                                           │
│      │    3. FIND FREE FRAME                                               │
│      │       - Check free list                                             │
│      │       - Or evict using LRU                                          │
│      │               │                                                      │
│      │               ▼                                                      │
│      │    4. DISK READ                                                      │
│      │       Read page from disk                                           │
│      │               │                                                      │
│      │               ▼                                                      │
│      │    5. LOAD INTO FRAME                                               │
│      │       Copy page data, update hash table                             │
│      │               │                                                      │
│      └───────┬───────┘                                                      │
│              ▼                                                              │
│   6. PIN PAGE                                                               │
│      Increment pin count                                                   │
│              │                                                              │
│              ▼                                                              │
│   7. RETURN TO QUERY                                                        │
│              │                                                              │
│              ▼                                                              │
│   8. UNPIN WHEN DONE                                                        │
│      Decrement pin count                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Page Replacement Policies

### 3.1 LRU (Least Recently Used)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     LRU REPLACEMENT                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Evict the page that was accessed longest ago                             │
│                                                                              │
│   Implementation: Doubly-linked list                                        │
│                                                                              │
│   Access page → Move to front                                              │
│   Eviction → Remove from back                                              │
│                                                                              │
│   MRU ←────────────────────────────────────────→ LRU                       │
│   ┌────┐   ┌────┐   ┌────┐   ┌────┐   ┌────┐   ┌────┐                     │
│   │ P5 │◄─►│ P2 │◄─►│ P8 │◄─►│ P1 │◄─►│ P9 │◄─►│ P3 │                     │
│   └────┘   └────┘   └────┘   └────┘   └────┘   └────┘                     │
│     ▲                                              │                        │
│     │                                              │                        │
│   Recently                                       Evict                      │
│   accessed                                       this                       │
│                                                                              │
│   Problem: Full table scans pollute cache                                  │
│   Solution: LRU-K, 2Q, CLOCK                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Clock Algorithm

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      CLOCK ALGORITHM                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Approximation of LRU with less overhead                                  │
│   Each frame has a "reference bit"                                         │
│                                                                              │
│   Circular buffer with clock hand:                                         │
│                                                                              │
│                        ┌────┐                                               │
│                   ┌────│ P1 │────┐                                         │
│                   │    │ r=1│    │                                         │
│              ┌────┐    └────┘    ┌────┐                                    │
│              │ P6 │              │ P2 │                                    │
│              │ r=0│     Clock    │ r=1│                                    │
│              └────┘     Hand     └────┘                                    │
│                   │      ↓       │                                         │
│              ┌────┐              ┌────┐                                    │
│              │ P5 │              │ P3 │                                    │
│              │ r=1│              │ r=0│                                    │
│              └────┘    ┌────┐    └────┘                                    │
│                   └────│ P4 │────┘                                         │
│                        │ r=1│                                               │
│                        └────┘                                               │
│                                                                              │
│   Algorithm:                                                                │
│   1. If current frame's ref bit = 0 → Evict it                            │
│   2. If ref bit = 1 → Set to 0, advance clock                             │
│   3. Repeat until found frame to evict                                    │
│                                                                              │
│   When page accessed → Set ref bit = 1                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 LRU-K and 2Q

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   ADVANCED REPLACEMENT POLICIES                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   LRU-K:                                                                    │
│   • Track last K references, not just last one                            │
│   • Evict page with oldest K-th reference                                 │
│   • Better for sequential scan resistance                                 │
│                                                                              │
│   2Q (Two Queue):                                                          │
│   ┌─────────────────────────────────────────────────────────┐              │
│   │ A1 Queue (FIFO) - First-time accessed pages            │              │
│   │ [P7] → [P5] → [P3] → [P1] → evict                      │              │
│   └─────────────────────────────────────────────────────────┘              │
│                    ↓ (if accessed again while in A1)                       │
│   ┌─────────────────────────────────────────────────────────┐              │
│   │ Am Queue (LRU) - Frequently accessed pages             │              │
│   │ [P8] ↔ [P2] ↔ [P6] ↔ [P4] → evict                     │              │
│   └─────────────────────────────────────────────────────────┘              │
│                                                                              │
│   • New pages go to A1 (short queue)                                       │
│   • If accessed again in A1 → promoted to Am                              │
│   • Protects frequently-used pages from scans                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Dirty Page Management

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DIRTY PAGE HANDLING                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   When a page is modified:                                                  │
│   1. Mark page as DIRTY in buffer frame                                    │
│   2. Write change to WAL (Write-Ahead Log) first                          │
│   3. Actual page write can be deferred                                    │
│                                                                              │
│   Page flushing strategies:                                                │
│                                                                              │
│   LAZY WRITE (Background Flusher)                                          │
│   • Background process periodically writes dirty pages                    │
│   • Reduces I/O spikes                                                    │
│   • PostgreSQL: bgwriter, checkpointer                                    │
│   • MySQL: page cleaner threads                                           │
│                                                                              │
│   CHECKPOINT                                                                │
│   • Periodically write ALL dirty pages                                    │
│   • Allows WAL truncation                                                 │
│   • Limits recovery time                                                  │
│                                                                              │
│   EVICTION FLUSH                                                           │
│   • When evicting dirty page, must write first                           │
│   • Synchronous I/O (slow if frequent)                                   │
│   • Try to pre-flush to avoid                                            │
│                                                                              │
│   Dirty page ratio:                                                        │
│   • Too high: Risk of I/O storm on checkpoint                            │
│   • Too low: Too much background I/O                                     │
│   • Typical target: 10-25% dirty pages                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Buffer Pool Configuration

### 5.1 PostgreSQL

```sql
-- Check shared buffers size
SHOW shared_buffers;  -- Default: 128MB

-- Recommended: 25% of RAM (up to ~8GB)
-- Set in postgresql.conf:
-- shared_buffers = '4GB'

-- Check buffer cache hit ratio
SELECT
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit)  as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
-- Target: > 99%

-- View buffer contents
CREATE EXTENSION pg_buffercache;
SELECT
    c.relname,
    count(*) AS buffers
FROM pg_buffercache b
JOIN pg_class c ON b.relfilenode = pg_relation_filenode(c.oid)
GROUP BY c.relname
ORDER BY buffers DESC
LIMIT 10;

-- Effective cache size (for planner)
SHOW effective_cache_size;  -- Include OS cache
```

### 5.2 MySQL/InnoDB

```sql
-- Check buffer pool size
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
-- Recommended: 70-80% of RAM for dedicated server

-- Buffer pool instances (for concurrency)
SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';
-- Recommended: 1 instance per GB of buffer pool

-- Check buffer pool statistics
SHOW STATUS LIKE 'Innodb_buffer_pool%';

-- Key metrics:
-- Innodb_buffer_pool_read_requests (logical reads)
-- Innodb_buffer_pool_reads (disk reads)
-- Hit ratio = 1 - (reads / read_requests)

SELECT
    (1 - (
        (SELECT Variable_value FROM performance_schema.global_status
         WHERE Variable_name = 'Innodb_buffer_pool_reads') /
        (SELECT Variable_value FROM performance_schema.global_status
         WHERE Variable_name = 'Innodb_buffer_pool_read_requests')
    )) * 100 AS buffer_pool_hit_ratio;
-- Target: > 99%

-- Warm up buffer pool on restart
SET GLOBAL innodb_buffer_pool_dump_at_shutdown = ON;
SET GLOBAL innodb_buffer_pool_load_at_startup = ON;
```

---

## 6. Other Caches

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      OTHER DATABASE CACHES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   QUERY CACHE (MySQL - Deprecated)                                          │
│   • Cached query results                                                   │
│   • Invalidated on any table modification                                  │
│   • Removed in MySQL 8.0 (too much contention)                            │
│                                                                              │
│   PLAN CACHE                                                                │
│   • Stores parsed and optimized query plans                               │
│   • Avoids re-parsing identical queries                                   │
│   • PostgreSQL: pg_prepared_statements                                    │
│   • MySQL: Query cache alternative via prepared statements                │
│                                                                              │
│   METADATA CACHE                                                           │
│   • Table definitions, column info                                        │
│   • PostgreSQL: relcache, syscache                                        │
│   • MySQL: table definition cache                                         │
│                                                                              │
│   OS PAGE CACHE                                                            │
│   • Operating system caches file reads                                    │
│   • PostgreSQL uses double buffering (can be wasteful)                   │
│   • InnoDB uses O_DIRECT to bypass OS cache                              │
│                                                                              │
│   CONNECTION CACHE / POOLING                                               │
│   • Reuse database connections                                            │
│   • PgBouncer, ProxySQL                                                   │
│   • Reduces connection overhead                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Monitoring and Tuning

```sql
-- PostgreSQL: Check for buffer issues
SELECT
    relname,
    heap_blks_read,
    heap_blks_hit,
    CASE WHEN heap_blks_hit + heap_blks_read > 0
        THEN round(100.0 * heap_blks_hit / (heap_blks_hit + heap_blks_read), 2)
        ELSE 0 END AS hit_ratio
FROM pg_statio_user_tables
ORDER BY heap_blks_read DESC
LIMIT 10;

-- MySQL: Monitor buffer pool
SELECT
    pool_id,
    pool_size,
    free_buffers,
    database_pages,
    modified_db_pages
FROM information_schema.innodb_buffer_pool_stats;

-- Signs you need more buffer pool:
-- • Hit ratio < 99%
-- • High disk I/O during normal operations
-- • Innodb_buffer_pool_reads increasing rapidly
```

---

## 8. Summary

| Component | Purpose | Key Setting |
|-----------|---------|-------------|
| Buffer Pool | Cache data pages | shared_buffers (PG), innodb_buffer_pool_size (MySQL) |
| Hash Table | Fast page lookup | Automatic |
| LRU/Clock | Page replacement | Automatic |
| Dirty List | Track modified pages | Background flushing |

**Key Metrics:**
- Buffer hit ratio > 99%
- Dirty page ratio: 10-25%
- Checkpoint frequency: Balance recovery vs I/O

**Sizing Guidelines:**
- PostgreSQL: 25% of RAM for shared_buffers
- MySQL: 70-80% of RAM for buffer pool
- Always leave memory for OS and connections
