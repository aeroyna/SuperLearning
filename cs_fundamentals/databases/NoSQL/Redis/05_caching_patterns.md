# Caching Patterns

## Learning Objectives
- Implement common caching strategies
- Design effective cache invalidation
- Handle cache failures gracefully
- Optimize cache hit rates and memory usage

---

## 1. Caching Fundamentals

### Why Caching?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Caching Benefits                                  │
│                                                                      │
│  Without Cache:                                                      │
│  ┌────────┐      ┌────────┐      ┌────────┐                         │
│  │ Client │─────▶│  App   │─────▶│   DB   │  Every request         │
│  │        │◀─────│        │◀─────│        │  hits database         │
│  └────────┘      └────────┘      └────────┘                         │
│                                   ~10-100ms                          │
│                                                                      │
│  With Cache:                                                         │
│  ┌────────┐      ┌────────┐      ┌────────┐      ┌────────┐        │
│  │ Client │─────▶│  App   │─────▶│ Redis  │      │   DB   │        │
│  │        │◀─────│        │◀─────│ Cache  │      │        │        │
│  └────────┘      └────────┘      └────────┘      └────────┘        │
│                                   ~1ms          (only on miss)       │
│                                                                      │
│  Benefits:                                                           │
│  • Reduce database load (80-99% of reads from cache)                │
│  • Lower latency (sub-millisecond vs milliseconds)                  │
│  • Higher throughput                                                │
│  • Cost savings (fewer database resources needed)                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Cache Hit Rate

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cache Effectiveness                               │
│                                                                      │
│  Hit Rate = Cache Hits / (Cache Hits + Cache Misses)                │
│                                                                      │
│  Target: 80-99% hit rate for most applications                      │
│                                                                      │
│  Factors affecting hit rate:                                         │
│  • TTL (Time To Live) - too short = more misses                     │
│  • Cache size - too small = evictions                               │
│  • Access patterns - random access = lower hit rate                 │
│  • Data volatility - frequent changes = more invalidations          │
│                                                                      │
│  Monitor in Redis:                                                   │
│  INFO stats                                                          │
│  keyspace_hits:1234567                                              │
│  keyspace_misses:12345                                              │
│  Hit rate = 1234567 / (1234567 + 12345) = 99%                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Cache-Aside (Lazy Loading)

### Pattern Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cache-Aside Pattern                               │
│                                                                      │
│  Read:                                                               │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  1. Check cache                                            │     │
│  │  2. If HIT: return cached data                            │     │
│  │  3. If MISS: read from database                           │     │
│  │  4. Store in cache                                        │     │
│  │  5. Return data                                           │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Write:                                                              │
│  ┌────────────────────────────────────────────────────────────┐     │
│  │  1. Write to database                                     │     │
│  │  2. Invalidate cache (or update cache)                    │     │
│  └────────────────────────────────────────────────────────────┘     │
│                                                                      │
│  Pros:                          Cons:                               │
│  ✓ Simple to implement          ✗ Cache miss penalty               │
│  ✓ Only caches accessed data    ✗ Potential stale data             │
│  ✓ Resilient to cache failure   ✗ Initial request slower           │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```python
import redis
import json

r = redis.Redis()

def get_user(user_id):
    cache_key = f"user:{user_id}"

    # 1. Try cache first
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)

    # 2. Cache miss - fetch from database
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)

    if user:
        # 3. Store in cache with TTL
        r.setex(cache_key, 3600, json.dumps(user))

    return user

def update_user(user_id, data):
    # 1. Update database
    db.query("UPDATE users SET ... WHERE id = %s", user_id)

    # 2. Invalidate cache
    r.delete(f"user:{user_id}")

    # Alternative: Update cache
    # r.setex(f"user:{user_id}", 3600, json.dumps(data))
```

### Cache Miss Stampede Prevention

```python
import time
import random

def get_with_lock(key, fetch_func, ttl=3600):
    """Prevent cache stampede with locking"""

    # Try cache
    cached = r.get(key)
    if cached:
        return json.loads(cached)

    # Acquire lock
    lock_key = f"lock:{key}"
    lock_acquired = r.set(lock_key, "1", nx=True, ex=10)

    if lock_acquired:
        try:
            # Double-check cache (another thread might have populated)
            cached = r.get(key)
            if cached:
                return json.loads(cached)

            # Fetch and cache
            data = fetch_func()
            r.setex(key, ttl, json.dumps(data))
            return data
        finally:
            r.delete(lock_key)
    else:
        # Wait for other thread to populate cache
        time.sleep(0.1)
        return get_with_lock(key, fetch_func, ttl)


def get_with_probabilistic_early_refresh(key, fetch_func, ttl=3600, beta=1):
    """Probabilistic early refresh to prevent stampede"""

    cached = r.get(key)
    if cached:
        data = json.loads(cached)
        remaining_ttl = r.ttl(key)

        # Probabilistic early refresh
        # As TTL decreases, probability of refresh increases
        if remaining_ttl > 0:
            delta = ttl - remaining_ttl
            if random.random() < beta * math.log(random.random()) * -delta:
                # Early refresh in background
                threading.Thread(target=lambda: refresh_cache(key, fetch_func, ttl)).start()

        return data

    # Cache miss
    return refresh_cache(key, fetch_func, ttl)
```

---

## 3. Write-Through

### Pattern Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Write-Through Pattern                             │
│                                                                      │
│  Write:                                                              │
│  ┌────────┐      ┌────────┐      ┌────────┐      ┌────────┐        │
│  │ Client │─────▶│  App   │─────▶│ Cache  │─────▶│   DB   │        │
│  │        │◀─────│        │◀─────│        │◀─────│        │        │
│  └────────┘      └────────┘      └────────┘      └────────┘        │
│                                                                      │
│  1. Application writes to cache                                     │
│  2. Cache writes to database synchronously                          │
│  3. Write confirmed only after DB write succeeds                   │
│                                                                      │
│  Pros:                          Cons:                               │
│  ✓ Cache always consistent      ✗ Higher write latency             │
│  ✓ Simple read path             ✗ Cache implementation complex     │
│  ✓ No stale data                ✗ Need reliable cache              │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation

```python
class WriteThroughCache:
    def __init__(self, redis_client, db):
        self.redis = redis_client
        self.db = db

    def get(self, key):
        cached = self.redis.get(key)
        if cached:
            return json.loads(cached)

        # Fallback to DB
        data = self.db.get(key)
        if data:
            self.redis.setex(key, 3600, json.dumps(data))
        return data

    def set(self, key, value, ttl=3600):
        # Write to DB first
        self.db.set(key, value)

        # Then update cache
        self.redis.setex(key, ttl, json.dumps(value))

    def delete(self, key):
        # Delete from DB first
        self.db.delete(key)

        # Then delete from cache
        self.redis.delete(key)
```

---

## 4. Write-Behind (Write-Back)

### Pattern Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Write-Behind Pattern                              │
│                                                                      │
│  Write:                                                              │
│  ┌────────┐      ┌────────┐      ┌────────┐                         │
│  │ Client │─────▶│  App   │─────▶│ Cache  │  (immediate return)     │
│  │        │◀─────│        │◀─────│        │                         │
│  └────────┘      └────────┘      └────────┘                         │
│                                       │                              │
│                                       │ async                        │
│                                       ▼                              │
│                                 ┌────────┐                           │
│                                 │   DB   │  (batched, delayed)      │
│                                 └────────┘                           │
│                                                                      │
│  Pros:                          Cons:                               │
│  ✓ Low write latency            ✗ Data loss risk                   │
│  ✓ Batch DB writes              ✗ Consistency delays               │
│  ✓ Absorbs write spikes         ✗ Complex implementation           │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation with Redis Streams

```python
import threading
import time

class WriteBehindCache:
    def __init__(self, redis_client, db):
        self.redis = redis_client
        self.db = db
        self.write_stream = "write_buffer"
        self._start_writer()

    def get(self, key):
        # Check pending writes first
        # Then check cache
        cached = self.redis.get(key)
        if cached:
            return json.loads(cached)
        return self.db.get(key)

    def set(self, key, value, ttl=3600):
        # Update cache immediately
        self.redis.setex(key, ttl, json.dumps(value))

        # Queue write to database
        self.redis.xadd(self.write_stream, {
            'operation': 'SET',
            'key': key,
            'value': json.dumps(value)
        })

    def _start_writer(self):
        def writer():
            last_id = '0'
            while True:
                entries = self.redis.xread(
                    {self.write_stream: last_id},
                    count=100,
                    block=1000
                )

                if entries:
                    batch = []
                    for stream, messages in entries:
                        for msg_id, data in messages:
                            batch.append(data)
                            last_id = msg_id

                    # Batch write to database
                    self._write_batch(batch)

                    # Trim processed entries
                    self.redis.xtrim(self.write_stream, maxlen=0)

        thread = threading.Thread(target=writer, daemon=True)
        thread.start()

    def _write_batch(self, batch):
        for item in batch:
            op = item[b'operation'].decode()
            key = item[b'key'].decode()

            if op == 'SET':
                value = json.loads(item[b'value'])
                self.db.set(key, value)
            elif op == 'DELETE':
                self.db.delete(key)
```

---

## 5. Cache Invalidation

### Strategies

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Invalidation Strategies                           │
│                                                                      │
│  1. TTL-Based (Time-Based Expiration)                               │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ SET key value EX 3600                                     │   │
│     │ • Simple, automatic cleanup                               │   │
│     │ • Eventual consistency                                    │   │
│     │ • Risk of serving stale data                              │   │
│     └──────────────────────────────────────────────────────────┘   │
│                                                                      │
│  2. Event-Driven (Pub/Sub)                                          │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ PUBLISH cache:invalidate user:123                        │   │
│     │ • Real-time invalidation                                  │   │
│     │ • Distributed cache coherence                             │   │
│     │ • Requires pub/sub infrastructure                         │   │
│     └──────────────────────────────────────────────────────────┘   │
│                                                                      │
│  3. Tag-Based                                                        │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ Tag entries, invalidate by tag                           │   │
│     │ SADD tag:user:123 "profile:123" "settings:123"           │   │
│     │ Invalidate all: DEL profile:123 settings:123             │   │
│     └──────────────────────────────────────────────────────────┘   │
│                                                                      │
│  4. Version-Based                                                    │
│     ┌──────────────────────────────────────────────────────────┐   │
│     │ Include version in key: user:123:v5                      │   │
│     │ Increment version to invalidate old data                 │   │
│     │ Old versions naturally expire                            │   │
│     └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Tag-Based Invalidation

```python
class TaggedCache:
    def __init__(self, redis_client):
        self.redis = redis_client

    def set(self, key, value, tags=None, ttl=3600):
        # Store value
        self.redis.setex(key, ttl, json.dumps(value))

        # Associate key with tags
        if tags:
            for tag in tags:
                self.redis.sadd(f"tag:{tag}", key)

    def get(self, key):
        cached = self.redis.get(key)
        return json.loads(cached) if cached else None

    def invalidate_by_tag(self, tag):
        tag_key = f"tag:{tag}"

        # Get all keys with this tag
        keys = self.redis.smembers(tag_key)

        if keys:
            # Delete all tagged keys
            self.redis.delete(*keys)

        # Delete tag set
        self.redis.delete(tag_key)


# Usage
cache = TaggedCache(redis)

# Cache user data with tags
cache.set("user:123:profile", profile_data, tags=["user:123", "profiles"])
cache.set("user:123:settings", settings_data, tags=["user:123", "settings"])

# Invalidate all user:123 data
cache.invalidate_by_tag("user:123")
```

### Distributed Invalidation

```python
# Publisher (when data changes)
def on_user_update(user_id):
    # Update database
    db.update_user(user_id, data)

    # Publish invalidation event
    redis.publish("cache:invalidate", json.dumps({
        "type": "user",
        "id": user_id
    }))


# Subscriber (on each app server)
def cache_invalidation_listener():
    pubsub = redis.pubsub()
    pubsub.subscribe("cache:invalidate")

    for message in pubsub.listen():
        if message["type"] == "message":
            event = json.loads(message["data"])

            if event["type"] == "user":
                local_cache.delete(f"user:{event['id']}:*")
```

---

## 6. Common Patterns

### Session Caching

```python
import uuid
import json

class SessionStore:
    def __init__(self, redis_client, ttl=3600):
        self.redis = redis_client
        self.ttl = ttl

    def create_session(self, user_id, data=None):
        session_id = str(uuid.uuid4())
        session_data = {
            "user_id": user_id,
            "created_at": time.time(),
            **(data or {})
        }

        self.redis.setex(
            f"session:{session_id}",
            self.ttl,
            json.dumps(session_data)
        )
        return session_id

    def get_session(self, session_id):
        data = self.redis.get(f"session:{session_id}")
        if data:
            # Extend TTL on access
            self.redis.expire(f"session:{session_id}", self.ttl)
            return json.loads(data)
        return None

    def destroy_session(self, session_id):
        self.redis.delete(f"session:{session_id}")

    def destroy_user_sessions(self, user_id):
        # Track sessions per user for bulk invalidation
        pattern = f"session:*"
        for key in self.redis.scan_iter(pattern):
            session = json.loads(self.redis.get(key))
            if session.get("user_id") == user_id:
                self.redis.delete(key)
```

### Rate Limiting

```python
def rate_limit_fixed_window(key, limit, window_seconds):
    """Fixed window rate limiting"""
    current = r.incr(key)

    if current == 1:
        r.expire(key, window_seconds)

    return current <= limit


def rate_limit_sliding_window(key, limit, window_seconds):
    """Sliding window with sorted sets"""
    now = time.time()
    window_start = now - window_seconds

    pipe = r.pipeline()

    # Remove old entries
    pipe.zremrangebyscore(key, 0, window_start)

    # Count requests in window
    pipe.zcard(key)

    # Add current request
    pipe.zadd(key, {str(now): now})

    # Set expiry
    pipe.expire(key, window_seconds)

    results = pipe.execute()
    request_count = results[1]

    if request_count >= limit:
        return False

    return True


def rate_limit_token_bucket(key, capacity, refill_rate):
    """Token bucket algorithm using Lua script"""
    lua_script = """
    local key = KEYS[1]
    local capacity = tonumber(ARGV[1])
    local refill_rate = tonumber(ARGV[2])
    local now = tonumber(ARGV[3])

    local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
    local tokens = tonumber(bucket[1]) or capacity
    local last_refill = tonumber(bucket[2]) or now

    -- Refill tokens
    local elapsed = now - last_refill
    local refill = elapsed * refill_rate
    tokens = math.min(capacity, tokens + refill)

    -- Try to consume token
    if tokens >= 1 then
        tokens = tokens - 1
        redis.call('HMSET', key, 'tokens', tokens, 'last_refill', now)
        redis.call('EXPIRE', key, capacity / refill_rate * 2)
        return 1
    else
        return 0
    end
    """
    return r.eval(lua_script, 1, key, capacity, refill_rate, time.time())
```

### Cache Warming

```python
def warm_cache():
    """Pre-populate cache on startup"""

    # Popular products
    products = db.query("SELECT * FROM products ORDER BY views DESC LIMIT 1000")
    for product in products:
        r.setex(f"product:{product.id}", 3600, json.dumps(product))

    # Active users
    users = db.query("SELECT * FROM users WHERE last_login > NOW() - INTERVAL '7 days'")
    for user in users:
        r.setex(f"user:{user.id}", 3600, json.dumps(user))


def warm_cache_lazy(key_pattern, fetch_func):
    """Warm cache in background as needed"""

    def wrapper(*args, **kwargs):
        key = key_pattern.format(*args, **kwargs)

        cached = r.get(key)
        if cached:
            return json.loads(cached)

        # Fetch and cache
        data = fetch_func(*args, **kwargs)
        r.setex(key, 3600, json.dumps(data))

        return data

    return wrapper
```

---

## 7. Memory Management

### Eviction Policies

```bash
# redis.conf

# Maximum memory
maxmemory 1gb

# Eviction policy
maxmemory-policy allkeys-lru    # Recommended for cache

# Options:
# noeviction     - Return error when memory limit reached
# allkeys-lru    - LRU among all keys
# volatile-lru   - LRU among keys with TTL
# allkeys-random - Random among all keys
# volatile-random- Random among keys with TTL
# volatile-ttl   - Evict keys with shortest TTL
# allkeys-lfu    - LFU among all keys (Redis 4.0+)
# volatile-lfu   - LFU among keys with TTL
```

### Memory Optimization

```redis
# Check memory usage
INFO memory
# used_memory_human: 1.5G
# maxmemory_human: 2G

# Per-key memory usage
MEMORY USAGE key

# Memory doctor
MEMORY DOCTOR

# Object encoding optimization
DEBUG OBJECT key
# encoding: ziplist (memory efficient)
# encoding: hashtable (fast but larger)
```

```python
# Best practices for memory efficiency

# Use short key names
# Bad:  user:profile:settings:notifications:email
# Good: u:1:p:s:n:e

# Use hashes for small objects (memory efficient encoding)
# Instead of: SET user:1:name "John", SET user:1:email "john@example.com"
# Use:        HSET user:1 name "John" email "john@example.com"

# Compress large values
import gzip

def set_compressed(key, value, ttl=3600):
    compressed = gzip.compress(json.dumps(value).encode())
    r.setex(key, ttl, compressed)

def get_compressed(key):
    data = r.get(key)
    if data:
        return json.loads(gzip.decompress(data))
    return None
```

---

## Summary

| Pattern | Latency | Consistency | Complexity |
|---------|---------|-------------|------------|
| Cache-Aside | Read: Low (hit), High (miss) | Eventual | Low |
| Write-Through | Write: Higher | Strong | Medium |
| Write-Behind | Write: Lowest | Eventual | High |

---

## Best Practices

```
✓ Set appropriate TTLs based on data volatility
✓ Monitor hit rates and optimize
✓ Implement cache stampede prevention
✓ Use consistent hashing for distributed cache
✓ Plan for cache failures (circuit breaker)
✓ Warm cache for predictable high-traffic events
✓ Use memory-efficient data structures
✓ Compress large values
✓ Choose right eviction policy for workload
✓ Monitor memory usage and set maxmemory
```
