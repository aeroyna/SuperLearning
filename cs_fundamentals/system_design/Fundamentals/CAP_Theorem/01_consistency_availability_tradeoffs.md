# Consistency vs Availability Trade-offs

Understanding when to prioritize consistency over availability (and vice versa) is critical for system design decisions.

---

## Consistency Levels

### Strong Consistency
After a write completes, all subsequent reads return that value.

```
Timeline:
T1: Write X=5 to Primary
T2: Write acknowledged
T3: Read X from ANY node → Returns 5 (guaranteed)
```

**Implementation**: Synchronous replication, distributed locks

### Eventual Consistency
If no new updates are made, eventually all reads will return the last updated value.

```
Timeline:
T1: Write X=5 to Primary
T2: Write acknowledged (async replication starts)
T3: Read X from Replica → May return old value (3)
T4: (replication completes)
T5: Read X from Replica → Returns 5
```

**Implementation**: Asynchronous replication

### Read-Your-Writes Consistency
A user always sees their own writes.

```python
class ReadYourWritesConsistency:
    def write(self, user_id, key, value):
        self.primary.write(key, value)
        self.cache.set(f"{user_id}:{key}:version", time.now())

    def read(self, user_id, key):
        last_write = self.cache.get(f"{user_id}:{key}:version")
        if last_write and time.now() - last_write < REPLICATION_LAG:
            return self.primary.read(key)  # Read from primary
        return self.replica.read(key)  # Safe to read from replica
```

### Monotonic Reads
Once a user reads a value, subsequent reads won't return older values.

### Causal Consistency
Causally related operations are seen in the same order by all nodes.

---

## When to Choose Consistency (CP)

### Use Cases Requiring Strong Consistency

| Domain | Reason |
|--------|--------|
| **Banking/Financial** | Account balances must be accurate |
| **Inventory Management** | Prevent overselling |
| **Booking Systems** | Prevent double-booking |
| **User Authentication** | Password changes must propagate immediately |
| **Leader Election** | Only one leader at a time |

### Example: Bank Transfer

```
Account A: $100
Account B: $0

Transfer $50 from A to B:
1. Read A balance: $100
2. Deduct from A: A = $50
3. Add to B: B = $50
4. Both writes must be atomic and consistent
```

If we used eventual consistency:
```
Node 1: A = $50, B = $50 (correct)
Node 2: A = $100, B = $50 (wrong! $50 created from nothing)
```

---

## When to Choose Availability (AP)

### Use Cases Where Eventual Consistency is Acceptable

| Domain | Reason |
|--------|--------|
| **Social Media Feeds** | Slight delay in posts appearing is acceptable |
| **Shopping Cart** | Better to show stale cart than error |
| **Analytics/Metrics** | Real-time accuracy not critical |
| **DNS** | Propagation delay is expected |
| **Session Storage** | Brief inconsistency acceptable |

### Example: Twitter Feed

```
User posts tweet at T1
Follower reads feed at T2 (1 second later):
- Eventual Consistency: May not see the tweet yet (acceptable)
- Strong Consistency: Would need to wait for all replicas (high latency)

For Twitter, slightly stale data is better than slow/unavailable feed
```

---

## Tunable Consistency

Many modern databases allow tuning consistency per operation.

### Cassandra Consistency Levels

```java
// Strong consistency (slower)
session.execute(
    SimpleStatement.builder("SELECT * FROM users WHERE id = ?")
        .setConsistencyLevel(ConsistencyLevel.QUORUM)
        .build()
);

// Eventual consistency (faster)
session.execute(
    SimpleStatement.builder("SELECT * FROM users WHERE id = ?")
        .setConsistencyLevel(ConsistencyLevel.ONE)
        .build()
);
```

### Quorum Formula

```
N = Total replicas
W = Write quorum (nodes that must acknowledge write)
R = Read quorum (nodes that must respond to read)

Strong Consistency: W + R > N

Example with N=3:
- W=2, R=2: Strong consistency (2+2 > 3)
- W=1, R=1: Eventual consistency (1+1 < 3)
```

---

## Hybrid Approaches

### Different Consistency for Different Data

```
User System:
- Authentication: Strong consistency (security critical)
- Profile data: Eventual consistency (less critical)
- Activity feed: Eventual consistency (acceptable delay)

E-commerce:
- Inventory count: Strong consistency (prevent overselling)
- Product reviews: Eventual consistency (delay acceptable)
- Recommendations: Eventual consistency (personalization)
```

### CQRS Pattern (Command Query Responsibility Segregation)

```
Commands (Writes) → Strongly consistent primary database
Queries (Reads)  → Eventually consistent read replicas

┌─────────────────┐
│   Commands      │ → Primary DB (strong consistency)
│   (writes)      │
└─────────────────┘
         ↓ (async replication)
┌─────────────────┐
│   Queries       │ → Read Replicas (eventual consistency)
│   (reads)       │
└─────────────────┘
```

---

## Interview Discussion Framework

```
1. Identify the data type and access pattern
2. Assess the cost of inconsistency:
   - Financial loss? (need strong)
   - User frustration? (assess severity)
   - Just cosmetic? (eventual is fine)
3. Assess the cost of unavailability:
   - Revenue loss per minute of downtime?
   - User experience impact?
4. Propose appropriate consistency level with justification
```

### Example Interview Answer

> "For the shopping cart, I'd use eventual consistency. Here's why:
> 1. The cost of showing a slightly stale cart is low
> 2. The cost of the cart being unavailable during a partition is high (lost sales)
> 3. We can handle conflicts by merging cart contents
> 4. Final checkout will use strong consistency to verify inventory"
