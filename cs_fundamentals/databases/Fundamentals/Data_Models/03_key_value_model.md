# Key-Value Model

## 1. Introduction

The **key-value model** is the simplest data model, storing data as a collection of key-value pairs. Each key is unique and maps to exactly one value, which can be a simple string, number, or complex serialized object.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KEY-VALUE MODEL STRUCTURE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────┬───────────────────────────────────────────┐   │
│   │          KEY            │                  VALUE                     │   │
│   ├─────────────────────────┼───────────────────────────────────────────┤   │
│   │  user:1001              │  {"name":"Alice","email":"alice@x.com"}   │   │
│   │  session:abc123def      │  {"user_id":1001,"expires":1705312800}    │   │
│   │  cache:product:42       │  {"name":"Widget","price":29.99}          │   │
│   │  counter:page_views     │  1547832                                   │   │
│   │  lock:resource:17       │  "node-1"                                  │   │
│   │  rate:user:1001         │  [timestamps...]                          │   │
│   └─────────────────────────┴───────────────────────────────────────────┘   │
│                                                                              │
│   Operations: GET, SET, DELETE, EXISTS                                       │
│   Time Complexity: O(1) for basic operations                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Concepts

### 2.1 Keys

Keys are unique identifiers, typically structured with namespaces:

```
# Key naming conventions
user:1001                    # Entity type : ID
session:abc123               # Resource : identifier
cache:api:products:42        # Hierarchical namespace
rate_limit:ip:192.168.1.1    # Feature : dimension : value
lock:order:processing:5001   # Distributed lock

# Best practices
- Use colons (:) or slashes (/) as separators
- Include type/namespace prefix
- Keep keys reasonably short (memory overhead)
- Use consistent naming across application
```

### 2.2 Values

Values can be various types depending on the database:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           VALUE TYPES                                        │
├────────────────┬────────────────────────────────────────────────────────────┤
│ String         │ "Hello World", "user:1001:profile" serialized JSON         │
│ Integer        │ 42, 1547832, -100                                          │
│ Float          │ 3.14159, 99.99                                             │
│ Binary         │ Images, serialized objects, protocol buffers              │
│ List           │ ["a", "b", "c"] - ordered collection                       │
│ Set            │ {"a", "b", "c"} - unique unordered collection              │
│ Hash           │ {field1: val1, field2: val2} - field-value pairs          │
│ Sorted Set     │ {(score1, member1), (score2, member2)} - scored members   │
└────────────────┴────────────────────────────────────────────────────────────┘
```

### 2.3 Time-To-Live (TTL)

Automatic expiration is a key feature:

```
SET session:abc123 "{...}" EX 3600      # Expires in 1 hour
SETEX cache:product:42 300 "{...}"      # Expires in 5 minutes
EXPIREAT key 1705312800                  # Expires at Unix timestamp

# Use cases for TTL
- Session management
- Cache invalidation
- Rate limiting windows
- Temporary locks
- Feature flags with auto-rollback
```

---

## 3. Redis Data Structures

Redis extends the basic key-value model with rich data structures:

### 3.1 Strings

```redis
# Basic operations
SET user:1001:name "Alice"
GET user:1001:name                       # "Alice"

# Atomic increment/decrement
SET counter:views 0
INCR counter:views                       # 1
INCRBY counter:views 10                  # 11
DECR counter:views                       # 10

# Set with expiration
SETEX session:abc 3600 '{"user_id":1001}'

# Set if not exists (for locks)
SETNX lock:resource:1 "node-1"           # Returns 1 if set, 0 if exists

# Multiple operations
MSET user:1:name "Alice" user:2:name "Bob"
MGET user:1:name user:2:name             # ["Alice", "Bob"]

# Bit operations
SETBIT user:1001:features 0 1            # Enable feature 0
GETBIT user:1001:features 0              # 1
BITCOUNT user:1001:features              # Count set bits
```

### 3.2 Lists

```redis
# Push operations
LPUSH queue:jobs "job1"                  # Push to left
RPUSH queue:jobs "job2"                  # Push to right

# Pop operations
LPOP queue:jobs                          # Pop from left
RPOP queue:jobs                          # Pop from right
BLPOP queue:jobs 30                      # Blocking pop with timeout

# Range operations
LRANGE queue:jobs 0 -1                   # Get all elements
LRANGE queue:jobs 0 9                    # Get first 10

# List as recent items
LPUSH user:1001:recent "page1"
LTRIM user:1001:recent 0 99              # Keep only last 100

# Length
LLEN queue:jobs
```

### 3.3 Sets

```redis
# Add members
SADD user:1001:tags "python" "redis" "docker"

# Remove members
SREM user:1001:tags "docker"

# Check membership
SISMEMBER user:1001:tags "python"        # 1 (true)

# Get all members
SMEMBERS user:1001:tags                  # ["python", "redis"]

# Set operations
SADD user:1002:tags "python" "java"
SINTER user:1001:tags user:1002:tags     # ["python"] - intersection
SUNION user:1001:tags user:1002:tags     # ["python", "redis", "java"]
SDIFF user:1001:tags user:1002:tags      # ["redis"] - in 1001 not in 1002

# Random member
SRANDMEMBER user:1001:tags               # Random tag
SPOP user:1001:tags                      # Pop random member
```

### 3.4 Sorted Sets (ZSets)

```redis
# Add with scores
ZADD leaderboard 100 "alice"
ZADD leaderboard 85 "bob" 92 "charlie"

# Get by rank (0-indexed)
ZRANGE leaderboard 0 2                   # Bottom 3: ["bob", "charlie", "alice"]
ZREVRANGE leaderboard 0 2                # Top 3: ["alice", "charlie", "bob"]
ZREVRANGE leaderboard 0 2 WITHSCORES     # With scores

# Get by score range
ZRANGEBYSCORE leaderboard 80 95          # Scores 80-95

# Get rank
ZRANK leaderboard "bob"                  # 0 (lowest score)
ZREVRANK leaderboard "alice"             # 0 (highest score)

# Increment score
ZINCRBY leaderboard 5 "bob"              # bob: 90

# Count in range
ZCOUNT leaderboard 80 95                 # Count with scores 80-95

# Remove
ZREM leaderboard "charlie"
ZREMRANGEBYRANK leaderboard 0 2          # Remove bottom 3
ZREMRANGEBYSCORE leaderboard 0 50        # Remove scores 0-50
```

### 3.5 Hashes

```redis
# Set fields
HSET user:1001 name "Alice" email "alice@x.com" age 28

# Get fields
HGET user:1001 name                      # "Alice"
HMGET user:1001 name email               # ["Alice", "alice@x.com"]
HGETALL user:1001                        # All fields and values

# Increment numeric fields
HINCRBY user:1001 age 1                  # 29
HINCRBYFLOAT user:1001 balance 10.50

# Check existence
HEXISTS user:1001 name                   # 1

# Delete fields
HDEL user:1001 age

# Get all keys or values
HKEYS user:1001                          # ["name", "email"]
HVALS user:1001                          # ["Alice", "alice@x.com"]
HLEN user:1001                           # 2
```

### 3.6 Streams

```redis
# Add to stream
XADD events * type "click" user "1001" page "/products"
# Returns: "1705312800000-0" (ID)

# Read from stream
XREAD COUNT 10 STREAMS events 0          # Read from beginning
XREAD BLOCK 5000 STREAMS events $        # Block for new messages

# Consumer groups
XGROUP CREATE events mygroup $ MKSTREAM
XREADGROUP GROUP mygroup consumer1 COUNT 10 STREAMS events >

# Acknowledge processing
XACK events mygroup "1705312800000-0"

# Range queries
XRANGE events - +                        # All messages
XRANGE events 1705312800000 1705316400000  # Time range
```

---

## 4. Common Patterns

### 4.1 Caching Pattern

```python
import redis
import json

r = redis.Redis()

def get_user(user_id):
    # Try cache first
    cache_key = f"cache:user:{user_id}"
    cached = r.get(cache_key)

    if cached:
        return json.loads(cached)

    # Cache miss - fetch from database
    user = database.query(f"SELECT * FROM users WHERE id = {user_id}")

    # Store in cache with TTL
    r.setex(cache_key, 300, json.dumps(user))  # 5 minute TTL

    return user

def invalidate_user_cache(user_id):
    r.delete(f"cache:user:{user_id}")

# Cache-aside with write-through
def update_user(user_id, data):
    # Update database
    database.update(f"UPDATE users SET ... WHERE id = {user_id}")

    # Invalidate cache (or update it)
    r.delete(f"cache:user:{user_id}")
```

### 4.2 Session Store

```python
import redis
import uuid
import json
from datetime import datetime

r = redis.Redis()
SESSION_TTL = 3600  # 1 hour

def create_session(user_id):
    session_id = str(uuid.uuid4())
    session_data = {
        'user_id': user_id,
        'created_at': datetime.utcnow().isoformat(),
        'last_active': datetime.utcnow().isoformat()
    }

    r.setex(f"session:{session_id}", SESSION_TTL, json.dumps(session_data))
    return session_id

def get_session(session_id):
    data = r.get(f"session:{session_id}")
    if not data:
        return None

    # Refresh TTL on access
    r.expire(f"session:{session_id}", SESSION_TTL)
    return json.loads(data)

def destroy_session(session_id):
    r.delete(f"session:{session_id}")
```

### 4.3 Rate Limiting

```python
import redis
import time

r = redis.Redis()

def is_rate_limited(user_id, limit=100, window=60):
    """
    Sliding window rate limiter
    Returns True if rate limited, False if allowed
    """
    key = f"rate:{user_id}"
    now = time.time()
    window_start = now - window

    pipe = r.pipeline()
    # Remove old entries
    pipe.zremrangebyscore(key, 0, window_start)
    # Count current window
    pipe.zcard(key)
    # Add current request
    pipe.zadd(key, {str(now): now})
    # Set expiry
    pipe.expire(key, window)

    results = pipe.execute()
    current_count = results[1]

    return current_count >= limit

# Token bucket alternative
def token_bucket_check(user_id, capacity=100, refill_rate=10):
    """
    Token bucket rate limiter
    capacity: max tokens
    refill_rate: tokens per second
    """
    key = f"bucket:{user_id}"
    now = time.time()

    # Get current state
    data = r.hgetall(key)

    if data:
        tokens = float(data[b'tokens'])
        last_update = float(data[b'last_update'])
        # Add tokens based on time passed
        tokens = min(capacity, tokens + (now - last_update) * refill_rate)
    else:
        tokens = capacity

    if tokens >= 1:
        tokens -= 1
        r.hset(key, mapping={'tokens': tokens, 'last_update': now})
        r.expire(key, 3600)
        return True  # Allowed

    return False  # Rate limited
```

### 4.4 Distributed Lock

```python
import redis
import uuid
import time

r = redis.Redis()

class DistributedLock:
    def __init__(self, name, timeout=10):
        self.name = f"lock:{name}"
        self.timeout = timeout
        self.token = str(uuid.uuid4())

    def acquire(self, blocking=True, block_timeout=None):
        start = time.time()
        while True:
            # Try to acquire lock
            if r.set(self.name, self.token, nx=True, ex=self.timeout):
                return True

            if not blocking:
                return False

            if block_timeout and (time.time() - start) >= block_timeout:
                return False

            time.sleep(0.1)

    def release(self):
        # Only release if we own the lock (Lua script for atomicity)
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        r.eval(lua_script, 1, self.name, self.token)

    def __enter__(self):
        self.acquire()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.release()

# Usage
with DistributedLock("order:process:1001"):
    # Critical section - only one process can execute this
    process_order(1001)
```

### 4.5 Pub/Sub Messaging

```python
import redis
import threading

r = redis.Redis()

# Publisher
def publish_event(channel, message):
    r.publish(channel, json.dumps(message))

# Subscriber
def subscribe_to_events(channels, handler):
    pubsub = r.pubsub()
    pubsub.subscribe(*channels)

    for message in pubsub.listen():
        if message['type'] == 'message':
            data = json.loads(message['data'])
            handler(message['channel'], data)

# Usage
def event_handler(channel, data):
    print(f"Received on {channel}: {data}")

# Start subscriber in background thread
thread = threading.Thread(
    target=subscribe_to_events,
    args=(['orders', 'payments'], event_handler)
)
thread.daemon = True
thread.start()

# Publish events
publish_event('orders', {'order_id': 1001, 'status': 'created'})
```

### 4.6 Leaderboard

```python
import redis

r = redis.Redis()

class Leaderboard:
    def __init__(self, name):
        self.key = f"leaderboard:{name}"

    def add_score(self, user_id, score):
        r.zadd(self.key, {user_id: score})

    def increment_score(self, user_id, amount):
        return r.zincrby(self.key, amount, user_id)

    def get_rank(self, user_id):
        rank = r.zrevrank(self.key, user_id)
        return rank + 1 if rank is not None else None

    def get_score(self, user_id):
        return r.zscore(self.key, user_id)

    def get_top(self, count=10):
        return r.zrevrange(self.key, 0, count - 1, withscores=True)

    def get_around_user(self, user_id, count=5):
        rank = r.zrevrank(self.key, user_id)
        if rank is None:
            return []
        start = max(0, rank - count)
        end = rank + count
        return r.zrevrange(self.key, start, end, withscores=True)

# Usage
lb = Leaderboard("weekly_game")
lb.add_score("player1", 1000)
lb.increment_score("player1", 50)
print(lb.get_top(10))
print(lb.get_rank("player1"))
```

---

## 5. Code Examples Across Languages

### Python (redis-py)

```python
import redis
from redis.cluster import RedisCluster

# Single instance
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Connection pool
pool = redis.ConnectionPool(host='localhost', port=6379, db=0)
r = redis.Redis(connection_pool=pool)

# Cluster
rc = RedisCluster(host='localhost', port=7000)

# Basic operations
r.set('key', 'value')
r.get('key')  # 'value'

# Pipeline (batch operations)
pipe = r.pipeline()
pipe.set('a', 1)
pipe.set('b', 2)
pipe.get('a')
pipe.get('b')
results = pipe.execute()  # [True, True, '1', '2']

# Transaction
with r.pipeline(transaction=True) as pipe:
    pipe.multi()
    pipe.incr('counter')
    pipe.incr('counter')
    results = pipe.execute()
```

### Java (Jedis)

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.Pipeline;
import redis.clients.jedis.Transaction;

public class RedisExample {
    private static JedisPool pool = new JedisPool("localhost", 6379);

    public static void main(String[] args) {
        try (Jedis jedis = pool.getResource()) {
            // Basic operations
            jedis.set("key", "value");
            String value = jedis.get("key");

            // Hash operations
            jedis.hset("user:1001", "name", "Alice");
            jedis.hset("user:1001", "email", "alice@example.com");
            Map<String, String> user = jedis.hgetAll("user:1001");

            // Pipeline
            Pipeline p = jedis.pipelined();
            p.set("a", "1");
            p.set("b", "2");
            p.get("a");
            p.get("b");
            List<Object> results = p.syncAndReturnAll();

            // Transaction
            Transaction t = jedis.multi();
            t.incr("counter");
            t.incr("counter");
            t.exec();
        }
    }
}
```

### JavaScript (ioredis)

```javascript
const Redis = require('ioredis');

// Single instance
const redis = new Redis({
    host: 'localhost',
    port: 6379
});

// Cluster
const cluster = new Redis.Cluster([
    { host: 'localhost', port: 7000 },
    { host: 'localhost', port: 7001 }
]);

async function examples() {
    // Basic operations
    await redis.set('key', 'value');
    const value = await redis.get('key');

    // With expiration
    await redis.setex('session', 3600, JSON.stringify({ userId: 1001 }));

    // Hash
    await redis.hset('user:1001', 'name', 'Alice', 'email', 'alice@example.com');
    const user = await redis.hgetall('user:1001');

    // Pipeline
    const results = await redis.pipeline()
        .set('a', 1)
        .set('b', 2)
        .get('a')
        .get('b')
        .exec();

    // Transaction
    const multi = redis.multi();
    multi.incr('counter');
    multi.incr('counter');
    await multi.exec();

    // Pub/Sub
    const sub = new Redis();
    sub.subscribe('notifications');
    sub.on('message', (channel, message) => {
        console.log(`${channel}: ${message}`);
    });

    await redis.publish('notifications', 'Hello!');
}

examples().catch(console.error);
```

---

## 6. Advantages and Limitations

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Speed** | O(1) operations, in-memory storage |
| **Simplicity** | Easy to understand and use |
| **Scalability** | Horizontal scaling via partitioning |
| **Flexibility** | Values can be any serialized data |
| **Atomic Operations** | Thread-safe primitives |
| **TTL Support** | Automatic expiration |

### Limitations

| Limitation | Description |
|------------|-------------|
| **No Complex Queries** | Only key-based access |
| **Memory Bound** | Data must fit in RAM |
| **No Relationships** | Cannot model connections between data |
| **Key Management** | Application must track key patterns |
| **Persistence Trade-offs** | Durability vs performance |

---

## 7. When to Use Key-Value Stores

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        USE CASES DECISION MATRIX                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ✓ USE KEY-VALUE                         ✗ DON'T USE KEY-VALUE             │
│   ─────────────────                       ─────────────────────              │
│   • Session storage                       • Complex queries                  │
│   • Caching                               • Reporting/analytics              │
│   • Rate limiting                         • Relational data                  │
│   • Leaderboards                          • Full-text search                 │
│   • Real-time counters                    • Graph traversals                 │
│   • Feature flags                         • Primary data store*              │
│   • Job queues                            • Transactions across keys**       │
│   • Pub/Sub messaging                                                        │
│                                                                              │
│   * Unless persistence is configured and acceptable                          │
│   ** Redis supports transactions but with limitations                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Summary

Key-value stores provide the fastest possible data access through simple key lookups. They are ideal for:

- **Caching layers** to reduce database load
- **Session management** with automatic expiration
- **Real-time features** requiring sub-millisecond latency
- **Distributed systems** needing coordination primitives

Redis extends the basic model with powerful data structures (lists, sets, sorted sets, hashes) that enable sophisticated use cases while maintaining O(1) or O(log N) performance.
