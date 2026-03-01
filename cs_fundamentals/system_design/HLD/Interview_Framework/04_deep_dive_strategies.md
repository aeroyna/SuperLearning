# Deep Dive Strategies

Techniques for diving deep into specific components during system design interviews.

## When to Deep Dive

The interviewer may ask you to deep dive when:
- They want to test your depth of knowledge
- A component is critical to the system
- They want to see how you handle complexity
- The high-level design is complete

### Signals to Deep Dive

```
Interviewer cues:
- "How would you implement the [X] component?"
- "Can you go deeper into [Y]?"
- "What's the internal design of [Z]?"
- "How does [W] scale?"
```

## Deep Dive Framework

### 1. State the Problem

Clearly articulate what the component needs to solve:

```
"The feed service needs to:
- Generate personalized feeds for 200M users
- Handle 500K feed requests per second
- Return results in under 200ms
- Support real-time updates"
```

### 2. Propose the Solution

Present your approach with clear reasoning:

```
"I'll use a hybrid push-pull model:
- Push: Pre-compute feeds for active users
- Pull: Generate on-demand for inactive users
- Why: Balances latency with compute costs"
```

### 3. Explain the Implementation

Go into technical details:

```
Data structures, algorithms, workflows:
- How data flows through the component
- What data structures are used
- How it handles edge cases
```

### 4. Discuss Trade-offs

Every design decision has trade-offs:

```
"This approach trades:
✅ Low latency for active users
❌ Higher storage costs
❌ Delayed updates for inactive users"
```

### 5. Address Scale and Reliability

Show how it works at scale:

```
- How to shard/partition
- How to handle failures
- How to monitor/debug
```

## Common Deep Dive Topics

### 1. Database Design

#### Schema Design
```sql
-- Example: Social network posts

CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content TEXT,
    media_urls JSON,
    created_at TIMESTAMP,
    INDEX idx_user_time (user_id, created_at)
);

-- Discuss:
- Why this schema?
- Index choices and trade-offs
- Partitioning strategy
- Read vs write optimization
```

#### Sharding Strategy
```python
# User-based sharding
def get_shard(user_id: int, num_shards: int = 1024) -> int:
    """Consistent hashing for user data."""
    return hash(user_id) % num_shards

# Time-based sharding for events
def get_time_shard(timestamp: datetime) -> str:
    """Partition by month for time-series data."""
    return f"events_{timestamp.strftime('%Y_%m')}"

# Discuss:
# - Why this partitioning scheme?
# - How to handle hot shards?
# - Cross-shard queries?
# - Rebalancing strategy?
```

#### Replication
```
Primary-Replica Setup:
┌─────────┐
│ Primary │◄── All writes
└────┬────┘
     │ Async replication
     ├─────────────────┐
     ▼                 ▼
┌─────────┐      ┌─────────┐
│Replica 1│      │Replica 2│◄── Read distribution
└─────────┘      └─────────┘

Discuss:
- Sync vs async replication
- Consistency guarantees
- Failover process
- Split-brain scenarios
```

### 2. Caching Strategy

#### Cache Architecture
```
Multi-Level Caching:

┌────────────────────────────────────────────┐
│               Application                   │
│  ┌────────────────────────────────────┐    │
│  │        L1: In-Process Cache        │    │
│  │      (100MB, 1ms, per-server)      │    │
│  └────────────────┬───────────────────┘    │
└───────────────────┼────────────────────────┘
                    │ miss
                    ▼
┌────────────────────────────────────────────┐
│           L2: Distributed Cache            │
│         (Redis, 10GB, 2-5ms, shared)       │
└────────────────────┬───────────────────────┘
                     │ miss
                     ▼
┌────────────────────────────────────────────┐
│              L3: Database                  │
│           (MySQL, 10-50ms, persistent)     │
└────────────────────────────────────────────┘
```

#### Cache Patterns
```python
# Cache-Aside (Lazy Loading)
async def get_user(user_id: str) -> User:
    # Try cache
    cached = await cache.get(f"user:{user_id}")
    if cached:
        return User.from_json(cached)

    # Cache miss - load from DB
    user = await db.get_user(user_id)

    # Populate cache
    await cache.set(f"user:{user_id}", user.to_json(), ttl=3600)

    return user

# Write-Through
async def update_user(user_id: str, data: dict) -> User:
    # Update DB first
    user = await db.update_user(user_id, data)

    # Then update cache
    await cache.set(f"user:{user_id}", user.to_json(), ttl=3600)

    return user

# Write-Behind (Write-Back)
async def update_user_async(user_id: str, data: dict) -> User:
    # Update cache immediately
    user = User(**data)
    await cache.set(f"user:{user_id}", user.to_json())

    # Queue DB write for async processing
    await write_queue.push({'user_id': user_id, 'data': data})

    return user
```

#### Cache Invalidation
```python
# Time-based invalidation
await cache.set(key, value, ttl=3600)  # Expire in 1 hour

# Event-based invalidation
async def on_user_update(user_id: str):
    await cache.delete(f"user:{user_id}")
    await cache.delete(f"user:{user_id}:profile")
    await cache.delete(f"user:{user_id}:feed")

# Version-based invalidation
async def get_with_version(key: str, version: int):
    versioned_key = f"{key}:v{version}"
    return await cache.get(versioned_key)
```

### 3. Message Queue Design

#### Queue Architecture
```
Producer-Consumer Pattern:

┌──────────┐     ┌───────────────────┐     ┌──────────┐
│ Producer │────►│   Message Queue   │────►│ Consumer │
│  (API)   │     │      (Kafka)      │     │ (Worker) │
└──────────┘     └───────────────────┘     └──────────┘

Topics & Partitions:
┌─────────────────────────────────────────────────────┐
│ Topic: user-events                                  │
│ ┌───────────┐ ┌───────────┐ ┌───────────┐          │
│ │Partition 0│ │Partition 1│ │Partition 2│          │
│ │ user 0-99 │ │user 100-199│ │user 200-299│         │
│ └───────────┘ └───────────┘ └───────────┘          │
└─────────────────────────────────────────────────────┘
```

#### Delivery Guarantees
```python
# At-most-once: Fire and forget
async def send_at_most_once(message):
    await queue.send(message)  # No retry on failure

# At-least-once: Retry until acknowledged
async def send_at_least_once(message):
    while True:
        try:
            await queue.send(message)
            return
        except Exception:
            await asyncio.sleep(1)

# Exactly-once: Idempotency + deduplication
async def process_exactly_once(message):
    # Check if already processed
    if await is_processed(message.id):
        return

    # Process with transaction
    async with db.transaction():
        await process(message)
        await mark_processed(message.id)
```

### 4. API Design

#### RESTful API
```yaml
# Resource-oriented design
POST   /api/v1/users              # Create user
GET    /api/v1/users/{id}         # Get user
PUT    /api/v1/users/{id}         # Update user
DELETE /api/v1/users/{id}         # Delete user

# Nested resources
GET    /api/v1/users/{id}/posts   # Get user's posts
POST   /api/v1/posts/{id}/likes   # Like a post

# Query parameters
GET    /api/v1/posts?limit=20&cursor=abc123

# Response format
{
    "data": [...],
    "meta": {
        "cursor": "xyz789",
        "has_more": true
    }
}
```

#### Rate Limiting
```python
class RateLimiter:
    """Token bucket rate limiter."""

    def __init__(self, rate: int, capacity: int):
        self.rate = rate          # Tokens per second
        self.capacity = capacity  # Max tokens

    async def allow(self, key: str) -> bool:
        now = time.time()

        # Get current bucket state
        bucket = await self.redis.hgetall(f"ratelimit:{key}")

        if not bucket:
            # New bucket
            await self.redis.hmset(f"ratelimit:{key}", {
                'tokens': self.capacity - 1,
                'last_refill': now
            })
            return True

        # Calculate tokens to add
        elapsed = now - float(bucket['last_refill'])
        tokens = min(
            self.capacity,
            float(bucket['tokens']) + elapsed * self.rate
        )

        if tokens >= 1:
            # Allow request
            await self.redis.hmset(f"ratelimit:{key}", {
                'tokens': tokens - 1,
                'last_refill': now
            })
            return True

        return False
```

### 5. Consistency Patterns

#### Distributed Transactions
```python
# Two-Phase Commit (2PC)
class TwoPhaseCommit:
    async def execute(self, participants, operation):
        # Phase 1: Prepare
        votes = []
        for p in participants:
            vote = await p.prepare(operation)
            votes.append(vote)

        if all(votes):
            # Phase 2: Commit
            for p in participants:
                await p.commit()
        else:
            # Rollback
            for p in participants:
                await p.rollback()

# Saga Pattern
class BookingSaga:
    async def execute(self):
        try:
            reservation = await hotel_service.reserve()
            payment = await payment_service.charge()
            confirmation = await confirmation_service.send()
        except Exception:
            # Compensating transactions
            await hotel_service.cancel(reservation)
            await payment_service.refund(payment)
            raise
```

#### Eventual Consistency
```python
# Read-your-writes consistency
class SessionConsistency:
    async def write(self, key, value):
        version = await db.write(key, value)
        session['last_write_version'] = version

    async def read(self, key):
        min_version = session.get('last_write_version', 0)
        return await db.read(key, min_version=min_version)

# Conflict resolution with vector clocks
class VectorClock:
    def __init__(self):
        self.clock = {}  # node_id -> counter

    def increment(self, node_id):
        self.clock[node_id] = self.clock.get(node_id, 0) + 1

    def merge(self, other):
        for node, count in other.clock.items():
            self.clock[node] = max(self.clock.get(node, 0), count)
```

### 6. Search & Indexing

#### Inverted Index
```python
class InvertedIndex:
    """Simple inverted index for full-text search."""

    def __init__(self):
        self.index = {}  # term -> set of doc_ids

    def index_document(self, doc_id: str, text: str):
        terms = self.tokenize(text)
        for term in terms:
            if term not in self.index:
                self.index[term] = set()
            self.index[term].add(doc_id)

    def search(self, query: str) -> List[str]:
        terms = self.tokenize(query)

        if not terms:
            return []

        # Intersection of all term results
        result = self.index.get(terms[0], set())
        for term in terms[1:]:
            result &= self.index.get(term, set())

        return list(result)

    def tokenize(self, text: str) -> List[str]:
        # Lowercase, remove punctuation, split
        return text.lower().split()
```

#### Search Ranking
```python
# TF-IDF scoring
def tf_idf(term, document, corpus):
    tf = document.count(term) / len(document)
    df = sum(1 for doc in corpus if term in doc)
    idf = math.log(len(corpus) / (1 + df))
    return tf * idf

# BM25 ranking
def bm25(query_terms, document, corpus, k1=1.5, b=0.75):
    score = 0
    avg_doc_len = sum(len(d) for d in corpus) / len(corpus)

    for term in query_terms:
        tf = document.count(term)
        df = sum(1 for doc in corpus if term in doc)
        idf = math.log((len(corpus) - df + 0.5) / (df + 0.5))

        numerator = tf * (k1 + 1)
        denominator = tf + k1 * (1 - b + b * len(document) / avg_doc_len)

        score += idf * numerator / denominator

    return score
```

## Deep Dive Examples

### Example 1: Feed Generation

```
Problem: Design Twitter's feed generation

Step 1: State the Problem
- 200M DAU requesting feeds
- 500K feed requests/second
- Each feed shows 500 tweets
- Real-time updates expected

Step 2: Propose Solution
Hybrid Push-Pull approach:
- Push: Pre-compute for active users
- Pull: On-demand for inactive users

Step 3: Implementation

Data Flow:
┌────────────┐     ┌────────────┐     ┌────────────┐
│   Tweet    │────►│   Fanout   │────►│   Feed     │
│   Created  │     │   Service  │     │   Cache    │
└────────────┘     └────────────┘     └────────────┘
                         │
                         ▼
                   ┌────────────┐
                   │ Celebrity  │
                   │   Handler  │ (Pull for high-follower users)
                   └────────────┘

Feed Generation:
1. User requests feed
2. Check cache for pre-computed feed
3. If miss: merge following tweets + celebrity tweets
4. Apply ranking algorithm
5. Return top 500

Step 4: Trade-offs
✅ Low latency for active users
✅ Handles celebrities without fanout explosion
❌ Storage overhead for cached feeds
❌ Slight delay for inactive users

Step 5: Scale
- Shard feed cache by user_id
- Celebrity tweets: separate hot tier
- Fanout workers: auto-scale on queue depth
```

### Example 2: Distributed Lock

```
Problem: Design a distributed lock service

Step 1: State the Problem
- Multiple services need exclusive access
- Must handle network partitions
- Locks should auto-expire
- High availability required

Step 2: Propose Solution
Redlock algorithm with multiple Redis nodes

Step 3: Implementation

Lock Acquisition:
1. Get current time T1
2. Try to acquire lock on N nodes
3. Calculate elapsed time
4. Lock acquired if majority (N/2+1) nodes AND
   time remaining > validity period

async def acquire_lock(resource, ttl):
    lock_id = generate_uuid()
    acquired = 0
    start = time.time()

    for node in redis_nodes:
        if await node.set(resource, lock_id, nx=True, px=ttl):
            acquired += 1

    elapsed = (time.time() - start) * 1000
    validity = ttl - elapsed - drift

    if acquired >= (len(redis_nodes) // 2 + 1) and validity > 0:
        return Lock(resource, lock_id, validity)
    else:
        # Release partial locks
        await release_lock(resource, lock_id)
        return None

Step 4: Trade-offs
✅ No single point of failure
✅ Correct even with clock drift
❌ Higher latency (multiple nodes)
❌ Requires odd number of nodes

Step 5: Scale & Reliability
- Use 5-7 Redis nodes across zones
- Monitor lock contention
- Implement fencing tokens for safety
```

## Tips for Deep Dives

### 1. Start Simple, Add Complexity
```
Don't: Jump into distributed system details
Do: "Start with single server, then scale..."
```

### 2. Use Concrete Numbers
```
Don't: "We need a lot of storage"
Do: "We need ~10TB for 1 year of data"
```

### 3. Draw Diagrams
```
Visual representations help:
- Data flow diagrams
- Component interactions
- State machines
```

### 4. Acknowledge Unknowns
```
"I'm not 100% sure about X, but my intuition is..."
"This would need testing to validate..."
```

### 5. Connect to Real Systems
```
"This is similar to how Kafka/Redis/Cassandra does it..."
"We could use existing solutions like..."
```

## Related Topics

- [[03_hld_steps|HLD Interview Steps]]
- [[../Case_Studies/00_case_studies|Case Studies]]
- [[../../Fundamentals/00_fundamentals|System Design Fundamentals]]

---

**Tags**: #system-design #hld #interview #deep-dive
