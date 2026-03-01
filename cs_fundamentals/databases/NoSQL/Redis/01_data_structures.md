# Redis Data Structures

## Learning Objectives
- Master all Redis data types and their operations
- Choose the right data structure for each use case
- Understand memory efficiency and performance characteristics
- Implement common patterns with appropriate structures

---

## 1. Strings

### Basic Operations

```redis
# SET and GET
SET user:1:name "Alice"
GET user:1:name                    # "Alice"

# SET with options
SET key value EX 3600              # Expire in 3600 seconds
SET key value PX 60000             # Expire in 60000 milliseconds
SET key value NX                   # Only if key doesn't exist
SET key value XX                   # Only if key exists

# SETNX - Atomic set-if-not-exists (for locks)
SETNX lock:resource "owner:123"

# SETEX - Set with expiration (atomic)
SETEX session:abc 3600 "session_data"

# Multiple keys
MSET key1 "val1" key2 "val2" key3 "val3"
MGET key1 key2 key3                # Returns array

# String length
STRLEN user:1:name                 # 5
```

### Numeric Operations

```redis
# Increment/Decrement
SET counter 10
INCR counter                       # 11
INCRBY counter 5                   # 16
DECR counter                       # 15
DECRBY counter 3                   # 12

# Float operations
SET price 19.99
INCRBYFLOAT price 0.01             # "20"
INCRBYFLOAT price -0.50            # "19.50"

# Use case: Rate limiting
INCR api:user:123:requests
EXPIRE api:user:123:requests 60    # Reset every minute
```

### String Manipulation

```redis
# Append
SET greeting "Hello"
APPEND greeting ", World!"         # "Hello, World!" (length 13)

# Range operations
GETRANGE greeting 0 4              # "Hello"
SETRANGE greeting 7 "Redis"        # "Hello, Redis!"

# Get and set atomically
GETSET counter 0                   # Returns old value, sets new

# GETEX - Get with expiration set
GETEX key EX 3600                  # Get and set 1 hour TTL
```

### Binary-Safe Strings

```redis
# Strings can store any binary data
SET image:1 "\x89PNG\r\n..."       # Store binary
SET serialized:obj "{\"json\":1}"  # Store JSON

# Common pattern: Serialize objects
SET user:1 '{"name":"Alice","age":30}'
```

---

## 2. Lists

### Basic Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis List Structure                             │
│                                                                      │
│  Implemented as doubly-linked list (quick push/pop at both ends)    │
│                                                                      │
│  LPUSH                                              RPUSH            │
│    │                                                  │              │
│    ▼                                                  ▼              │
│  ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐         │
│  │  A  │◀─▶│  B  │◀─▶│  C  │◀─▶│  D  │◀─▶│  E  │◀─▶│  F  │         │
│  └─────┘   └─────┘   └─────┘   └─────┘   └─────┘   └─────┘         │
│    ▲                                                  ▲              │
│    │                                                  │              │
│  LPOP                                               RPOP             │
│                                                                      │
│  Index:  0         1         2         3         4         5        │
│         -6        -5        -4        -3        -2        -1        │
└─────────────────────────────────────────────────────────────────────┘
```

```redis
# Push elements
LPUSH tasks "task1"                # Push to left (head)
RPUSH tasks "task2"                # Push to right (tail)
LPUSH tasks "task0" "task-1"       # Push multiple

# Pop elements
LPOP tasks                         # Pop from left
RPOP tasks                         # Pop from right
LPOP tasks 3                       # Pop 3 elements (Redis 6.2+)

# Blocking pop (for queues)
BLPOP queue1 queue2 30             # Block 30 seconds waiting
BRPOP queue 0                      # Block indefinitely

# Range (0-indexed, inclusive)
LRANGE tasks 0 -1                  # All elements
LRANGE tasks 0 9                   # First 10 elements
LRANGE tasks -3 -1                 # Last 3 elements
```

### List Operations

```redis
# Length
LLEN tasks                         # Number of elements

# Index access
LINDEX tasks 0                     # Get first element
LINDEX tasks -1                    # Get last element

# Set by index
LSET tasks 0 "new_first_task"

# Insert relative to pivot
LINSERT tasks BEFORE "task2" "task1.5"
LINSERT tasks AFTER "task2" "task2.5"

# Remove elements
LREM tasks 0 "task1"               # Remove all "task1"
LREM tasks 2 "task1"               # Remove first 2 "task1"
LREM tasks -2 "task1"              # Remove last 2 "task1"

# Trim list
LTRIM tasks 0 99                   # Keep only first 100

# Move between lists
LMOVE source dest LEFT RIGHT       # Pop from source, push to dest
BLMOVE source dest LEFT RIGHT 30   # Blocking version
```

### Use Cases

```redis
# Message Queue
LPUSH queue:emails "email_job_1"
BRPOP queue:emails 0               # Workers consume

# Recent Items (capped list)
LPUSH user:1:recent_views "product:123"
LTRIM user:1:recent_views 0 9      # Keep last 10

# Activity Feed
LPUSH feed:user:1 "User X liked your post"
LRANGE feed:user:1 0 19            # Get last 20 activities
```

---

## 3. Sets

### Basic Operations

```redis
# Add members
SADD tags:post:1 "redis" "database" "nosql"

# Remove members
SREM tags:post:1 "nosql"

# Check membership
SISMEMBER tags:post:1 "redis"      # 1 (true) or 0 (false)
SMISMEMBER tags:post:1 "redis" "sql" # [1, 0]

# Get all members
SMEMBERS tags:post:1               # All members (small sets only)

# Cardinality (count)
SCARD tags:post:1                  # Number of members

# Random members
SRANDMEMBER tags:post:1            # One random member
SRANDMEMBER tags:post:1 3          # 3 random members
SPOP tags:post:1                   # Remove and return random
```

### Set Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Set Operations                                    │
│                                                                      │
│  Set A: {1, 2, 3, 4}          Set B: {3, 4, 5, 6}                   │
│                                                                      │
│  SUNION A B:                  SINTER A B:                           │
│  ┌─────────────────┐          ┌─────────────────┐                   │
│  │ {1,2,3,4,5,6}   │          │    {3, 4}       │                   │
│  └─────────────────┘          └─────────────────┘                   │
│                                                                      │
│  SDIFF A B:                   SDIFF B A:                            │
│  ┌─────────────────┐          ┌─────────────────┐                   │
│  │    {1, 2}       │          │    {5, 6}       │                   │
│  │  (in A not B)   │          │  (in B not A)   │                   │
│  └─────────────────┘          └─────────────────┘                   │
└─────────────────────────────────────────────────────────────────────┘
```

```redis
# Union
SUNION set1 set2 set3              # All members from all sets
SUNIONSTORE dest set1 set2         # Store result

# Intersection
SINTER set1 set2                   # Common members
SINTERSTORE dest set1 set2         # Store result
SINTERCARD 2 set1 set2 LIMIT 100   # Count intersection (Redis 7+)

# Difference
SDIFF set1 set2                    # Members in set1 not in set2
SDIFFSTORE dest set1 set2          # Store result

# Move member
SMOVE source dest member           # Atomic move
```

### Use Cases

```redis
# Tagging system
SADD tags:python "article:1" "article:5" "article:9"
SADD tags:redis "article:1" "article:3"
SINTER tags:python tags:redis      # Articles with both tags

# Unique visitors
SADD visitors:2024-01-15 "user:123" "user:456"
SCARD visitors:2024-01-15          # Count unique visitors

# Online users
SADD online:users "user:1" "user:2"
SISMEMBER online:users "user:1"    # Check if online

# Social features
SADD friends:user:1 "user:2" "user:3" "user:4"
SADD friends:user:2 "user:1" "user:3" "user:5"
SINTER friends:user:1 friends:user:2  # Mutual friends
```

---

## 4. Sorted Sets

### Basic Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Sorted Set Structure                              │
│                                                                      │
│  Members sorted by score (ascending)                                 │
│                                                                      │
│  Score:    100      200       350       400       500               │
│            │         │         │         │         │                 │
│            ▼         ▼         ▼         ▼         ▼                 │
│         ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                 │
│         │Alice│  │ Bob │  │Carol│  │David│  │ Eve │                 │
│         └─────┘  └─────┘  └─────┘  └─────┘  └─────┘                 │
│                                                                      │
│  Rank:     0        1         2         3         4                  │
│  (0-indexed, by score ascending)                                     │
└─────────────────────────────────────────────────────────────────────┘
```

```redis
# Add with scores
ZADD leaderboard 100 "alice" 200 "bob" 150 "carol"

# Add options
ZADD key NX score member          # Only if not exists
ZADD key XX score member          # Only if exists
ZADD key GT score member          # Only if new score > current
ZADD key LT score member          # Only if new score < current

# Get score and rank
ZSCORE leaderboard "alice"         # 100
ZRANK leaderboard "alice"          # Rank (0-indexed, low to high)
ZREVRANK leaderboard "alice"       # Rank (high to low)

# Increment score
ZINCRBY leaderboard 50 "alice"     # 150

# Remove members
ZREM leaderboard "alice"
ZREMRANGEBYRANK leaderboard 0 9    # Remove lowest 10
ZREMRANGEBYSCORE leaderboard 0 100 # Remove scores 0-100
```

### Range Queries

```redis
# By rank
ZRANGE leaderboard 0 9             # Top 10 (lowest scores)
ZRANGE leaderboard 0 9 WITHSCORES  # With scores
ZREVRANGE leaderboard 0 9          # Top 10 (highest scores)

# By score
ZRANGEBYSCORE leaderboard 100 200           # Score 100-200
ZRANGEBYSCORE leaderboard (100 200          # Score 100< x <=200
ZRANGEBYSCORE leaderboard -inf +inf         # All
ZRANGEBYSCORE leaderboard 100 200 LIMIT 0 5 # With pagination

# Count in range
ZCOUNT leaderboard 100 200         # Count scores 100-200
ZCARD leaderboard                  # Total members
ZLEXCOUNT key [a [z               # Lexicographic count
```

### Set Operations on Sorted Sets

```redis
# Union with weight
ZUNIONSTORE dest 2 zset1 zset2 WEIGHTS 1 2
# Resulting score = score1*1 + score2*2

# Intersection
ZINTERSTORE dest 2 zset1 zset2 AGGREGATE MIN
# AGGREGATE: SUM (default), MIN, MAX

# Range store (Redis 6.2+)
ZRANGESTORE dest src 0 9 BYSCORE
```

### Use Cases

```redis
# Leaderboard
ZADD game:leaderboard 1500 "player:1" 2300 "player:2"
ZINCRBY game:leaderboard 100 "player:1"       # Add points
ZREVRANGE game:leaderboard 0 9 WITHSCORES     # Top 10

# Priority Queue
ZADD jobs:priority 1 "critical_job"
ZADD jobs:priority 5 "normal_job"
ZADD jobs:priority 10 "low_priority_job"
ZPOPMIN jobs:priority                          # Get highest priority

# Time-based events
ZADD events 1705344000 "event:1"               # Unix timestamp as score
ZRANGEBYSCORE events 1705340000 1705350000     # Events in time range

# Rate limiting with sliding window
ZADD rate:user:123 1705344000.123 "req1"
ZREMRANGEBYSCORE rate:user:123 0 (now-60)      # Remove old
ZCARD rate:user:123                             # Count in window
```

---

## 5. Hashes

### Basic Operations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Hash Structure                                    │
│                                                                      │
│  Key: user:1                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Field        │  Value                                      │    │
│  ├───────────────┼─────────────────────────────────────────────┤    │
│  │  name         │  "John Doe"                                 │    │
│  │  email        │  "john@example.com"                         │    │
│  │  age          │  "30"                                       │    │
│  │  created_at   │  "2024-01-15"                               │    │
│  │  login_count  │  "42"                                       │    │
│  └───────────────┴─────────────────────────────────────────────┘    │
│                                                                      │
│  • Efficient for objects with many fields                           │
│  • Fields and values are strings                                    │
│  • Memory efficient (special encoding for small hashes)             │
└─────────────────────────────────────────────────────────────────────┘
```

```redis
# Set fields
HSET user:1 name "John Doe" email "john@example.com"
HSETNX user:1 created_at "2024-01-15"  # Only if not exists

# Get fields
HGET user:1 name                    # Single field
HMGET user:1 name email age         # Multiple fields
HGETALL user:1                      # All fields and values

# Check existence
HEXISTS user:1 email                # 1 or 0
HLEN user:1                         # Number of fields

# Delete fields
HDEL user:1 temp_field

# Increment
HINCRBY user:1 login_count 1
HINCRBYFLOAT user:1 balance 10.50
```

### Scanning and Iteration

```redis
# Get all keys or values
HKEYS user:1                        # All field names
HVALS user:1                        # All values

# Iterate (for large hashes)
HSCAN user:1 0 MATCH login* COUNT 10

# Random field
HRANDFIELD user:1 3                 # 3 random fields
HRANDFIELD user:1 3 WITHVALUES      # With values
```

### Use Cases

```redis
# User profile
HSET user:1000 name "Alice" email "alice@example.com" age 28

# Session data
HSET session:abc user_id 1000 ip "192.168.1.1" login_time 1705344000
EXPIRE session:abc 3600

# Shopping cart
HSET cart:user:1 product:1 2 product:5 1 product:10 3
HINCRBY cart:user:1 product:1 1      # Add one more
HGETALL cart:user:1                  # Get cart contents

# Configuration
HSET config:app feature:dark_mode 1 feature:beta 0
```

---

## 6. Specialized Types

### Bitmaps

```redis
# Bitmaps are strings with bit-level operations
SETBIT logins:user:1:2024-01 0 1    # Day 1 logged in
SETBIT logins:user:1:2024-01 4 1    # Day 5 logged in
GETBIT logins:user:1:2024-01 0      # Check day 1

# Count bits
BITCOUNT logins:user:1:2024-01      # Days logged in

# Bit operations
BITOP AND result key1 key2          # AND operation
BITOP OR result key1 key2           # OR operation
BITOP XOR result key1 key2          # XOR operation
BITOP NOT result key1               # NOT operation

# Find first bit
BITPOS logins:user:1:2024-01 1      # First day logged in
BITPOS logins:user:1:2024-01 0      # First day not logged in

# Use case: Track daily active users
SETBIT dau:2024-01-15 12345 1       # User 12345 active
BITCOUNT dau:2024-01-15             # Count DAU
BITOP OR mau:2024-01 dau:2024-01-* # Monthly active
```

### HyperLogLog

```redis
# Probabilistic cardinality counting (0.81% error)
# Uses only 12KB regardless of cardinality

PFADD unique:visitors "user1" "user2" "user3"
PFADD unique:visitors "user1" "user4"  # Duplicates ignored
PFCOUNT unique:visitors                # ~4

# Merge HyperLogLogs
PFMERGE total:visitors daily:day1 daily:day2

# Use case: Unique visitors per page
PFADD page:home:visitors "ip1" "ip2" "ip3"
PFADD page:about:visitors "ip1" "ip4"
PFCOUNT page:home:visitors page:about:visitors  # Total unique
```

### Geospatial

```redis
# Add locations
GEOADD locations -73.935242 40.730610 "New York"
GEOADD locations -122.419416 37.774929 "San Francisco"
GEOADD locations -0.127758 51.507351 "London"

# Get coordinates
GEOPOS locations "New York"

# Distance between points
GEODIST locations "New York" "San Francisco" km  # ~4129 km

# Find nearby (radius search)
GEORADIUS locations -73.9 40.7 100 km WITHDIST
GEOSEARCH locations FROMMEMBER "New York" BYRADIUS 500 km

# GeoHash
GEOHASH locations "New York"        # Returns geohash string

# Use case: Find nearby stores
GEOADD stores -122.4194 37.7749 "store:1"
GEOSEARCH stores FROMLONLAT -122.42 37.78 BYRADIUS 5 km ASC
```

---

## 7. Memory Optimization

### Encoding Types

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Internal Encodings                                │
│                                                                      │
│  Type          Small Size Encoding       Large Size Encoding        │
│  ────────────  ─────────────────────     ─────────────────────      │
│  String        int, embstr              raw                         │
│  List          listpack                 quicklist                   │
│  Set           intset, listpack         hashtable                   │
│  Sorted Set    listpack                 skiplist + hashtable        │
│  Hash          listpack                 hashtable                   │
│                                                                      │
│  Small encodings are more memory efficient                          │
│  Thresholds configurable in redis.conf                              │
└─────────────────────────────────────────────────────────────────────┘
```

```redis
# Check encoding
DEBUG OBJECT key

# Memory usage
MEMORY USAGE key
MEMORY DOCTOR
```

### Best Practices

```
✓ Use hashes for objects instead of many string keys
  Bad:  SET user:1:name "John", SET user:1:email "..."
  Good: HSET user:1 name "John" email "..."

✓ Use short key names in production
  Bad:  user:profile:settings:notifications:email
  Good: u:1:s:n:e

✓ Set appropriate TTLs to avoid memory bloat
  EXPIRE key 3600

✓ Use SCAN instead of KEYS in production
  Bad:  KEYS user:*
  Good: SCAN 0 MATCH user:* COUNT 100

✓ Choose appropriate data structure
  Counting: HyperLogLog
  Membership: Set
  Ranking: Sorted Set
  Queue: List
  Object: Hash
```

---

## Summary

| Data Type | Use Case | Time Complexity |
|-----------|----------|-----------------|
| String | Cache, counters, locks | O(1) |
| List | Queues, stacks, feeds | O(1) push/pop, O(N) range |
| Set | Tags, unique items | O(1) add/remove/check |
| Sorted Set | Leaderboards, priority queues | O(log N) |
| Hash | Objects, profiles | O(1) per field |
| Bitmap | Daily active, feature flags | O(1) per bit |
| HyperLogLog | Unique counting | O(1) |
| Geo | Location queries | O(log N) |
