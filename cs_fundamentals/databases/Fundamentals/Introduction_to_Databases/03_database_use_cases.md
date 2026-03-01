# Database Use Cases

## 1. Matching Databases to Requirements

Choosing the right database is one of the most critical architectural decisions. This guide covers real-world scenarios and the rationale behind database selection.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DATABASE SELECTION CRITERIA                       │
├─────────────────────────────────────────────────────────────────────┤
│  1. Data Model    - Structure of your data                          │
│  2. Query Patterns - How you'll access data                         │
│  3. Scale         - Volume and velocity of data                     │
│  4. Consistency   - ACID vs eventual consistency                    │
│  5. Availability  - Uptime requirements                             │
│  6. Latency       - Response time needs                             │
│  7. Team Skills   - Existing expertise                              │
│  8. Ecosystem     - Tools, libraries, community                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. RDBMS Use Cases

### 2.1 E-Commerce Platform

```sql
-- Order management requires ACID guarantees
BEGIN TRANSACTION;

-- Deduct inventory
UPDATE products
SET stock = stock - 1
WHERE product_id = 1001 AND stock > 0;

-- Create order
INSERT INTO orders (customer_id, product_id, quantity, total)
VALUES (500, 1001, 1, 29.99);

-- Process payment
INSERT INTO payments (order_id, amount, status)
VALUES (LAST_INSERT_ID(), 29.99, 'completed');

COMMIT;
```

**Why RDBMS:**
- Strong consistency for inventory and payments
- Complex queries for reporting and analytics
- Referential integrity between orders, products, customers
- ACID transactions prevent overselling

**Recommended:** PostgreSQL, MySQL

### 2.2 Banking and Financial Systems

```sql
-- Money transfer must be atomic
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 1000
    WHERE account_id = 'ACC001' AND balance >= 1000;

    UPDATE accounts SET balance = balance + 1000
    WHERE account_id = 'ACC002';

    INSERT INTO transactions (from_acc, to_acc, amount, timestamp)
    VALUES ('ACC001', 'ACC002', 1000, NOW());
COMMIT;

-- Audit trail query
SELECT t.*, a1.holder as from_holder, a2.holder as to_holder
FROM transactions t
JOIN accounts a1 ON t.from_acc = a1.account_id
JOIN accounts a2 ON t.to_acc = a2.account_id
WHERE t.timestamp BETWEEN '2024-01-01' AND '2024-01-31'
ORDER BY t.timestamp DESC;
```

**Why RDBMS:**
- Absolute consistency (money can't disappear)
- Audit requirements (complete transaction history)
- Complex regulatory compliance queries
- Decades of proven reliability

**Recommended:** Oracle, PostgreSQL, SQL Server

### 2.3 Content Management System (CMS)

```sql
CREATE TABLE articles (
    id INT PRIMARY KEY,
    title VARCHAR(500),
    slug VARCHAR(500) UNIQUE,
    content TEXT,
    author_id INT REFERENCES users(id),
    category_id INT REFERENCES categories(id),
    status ENUM('draft', 'published', 'archived'),
    published_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE article_tags (
    article_id INT REFERENCES articles(id),
    tag_id INT REFERENCES tags(id),
    PRIMARY KEY (article_id, tag_id)
);

-- Complex content query
SELECT a.*, u.name as author, c.name as category,
       GROUP_CONCAT(t.name) as tags
FROM articles a
JOIN users u ON a.author_id = u.id
LEFT JOIN categories c ON a.category_id = c.id
LEFT JOIN article_tags at ON a.id = at.article_id
LEFT JOIN tags t ON at.tag_id = t.id
WHERE a.status = 'published'
GROUP BY a.id
ORDER BY a.published_at DESC
LIMIT 10;
```

**Recommended:** PostgreSQL (with full-text search), MySQL

---

## 3. Document Database Use Cases

### 3.1 Product Catalog

```javascript
// MongoDB - Flexible product attributes
db.products.insertOne({
    _id: ObjectId(),
    name: "Gaming Laptop",
    brand: "TechBrand",
    price: 1299.99,
    category: "Electronics",
    attributes: {
        processor: "Intel i7-12700H",
        ram: "16GB DDR5",
        storage: "512GB NVMe SSD",
        display: "15.6 inch 144Hz",
        gpu: "RTX 3060"
    },
    variants: [
        { sku: "GL-16-512", ram: "16GB", storage: "512GB", price: 1299.99 },
        { sku: "GL-32-1TB", ram: "32GB", storage: "1TB", price: 1599.99 }
    ],
    reviews: [
        { user: "john123", rating: 5, comment: "Great laptop!", date: ISODate() }
    ]
});

// Query with nested attributes
db.products.find({
    category: "Electronics",
    "attributes.ram": { $regex: /^16GB/ },
    price: { $lt: 1500 }
});
```

**Why Document DB:**
- Products have varying attributes (laptop vs shirt vs book)
- Schema evolves frequently with new product types
- Embedded reviews avoid expensive joins
- Natural fit for JSON APIs

**Recommended:** MongoDB, CouchDB

### 3.2 User Profiles and Preferences

```javascript
// Complex user profile with preferences
{
    _id: ObjectId("user123"),
    username: "john_doe",
    email: "john@example.com",
    profile: {
        firstName: "John",
        lastName: "Doe",
        avatar: "https://cdn.example.com/avatars/john.jpg",
        bio: "Software developer and coffee enthusiast"
    },
    preferences: {
        theme: "dark",
        language: "en-US",
        notifications: {
            email: true,
            push: false,
            sms: true,
            digest: "weekly"
        },
        privacy: {
            profileVisibility: "friends",
            showOnlineStatus: false
        }
    },
    socialLinks: [
        { platform: "twitter", url: "https://twitter.com/johndoe" },
        { platform: "github", url: "https://github.com/johndoe" }
    ],
    activity: {
        lastLogin: ISODate("2024-01-15T10:30:00Z"),
        loginCount: 247
    }
}
```

**Why Document DB:**
- User preferences vary widely
- Fast reads of entire user profile
- Easy to add new preference fields
- Schema-less allows experimentation

**Recommended:** MongoDB, Amazon DocumentDB

### 3.3 Real-Time Collaboration (Google Docs-like)

```javascript
// Document with version history
{
    _id: ObjectId("doc123"),
    title: "Project Proposal",
    content: { /* Operational Transformation or CRDT data */ },
    collaborators: [
        { userId: "user1", role: "owner", permissions: ["read", "write", "share"] },
        { userId: "user2", role: "editor", permissions: ["read", "write"] }
    ],
    versions: [
        { version: 1, timestamp: ISODate(), changes: [...], author: "user1" },
        { version: 2, timestamp: ISODate(), changes: [...], author: "user2" }
    ],
    comments: [
        { id: "c1", position: { start: 10, end: 20 }, text: "Review this", author: "user2" }
    ]
}
```

**Recommended:** MongoDB with change streams, CouchDB for offline-first

---

## 4. Key-Value Store Use Cases

### 4.1 Session Management

```python
import redis

r = redis.Redis(host='localhost', port=6379)

# Store session
session_data = {
    "user_id": "12345",
    "username": "john_doe",
    "role": "admin",
    "cart": json.dumps(["item1", "item2"]),
    "last_active": datetime.now().isoformat()
}
r.hset("session:abc123", mapping=session_data)
r.expire("session:abc123", 3600)  # 1 hour TTL

# Retrieve session
session = r.hgetall("session:abc123")

# Check session exists
if r.exists("session:abc123"):
    r.expire("session:abc123", 3600)  # Extend TTL
```

**Why Key-Value:**
- Sub-millisecond reads
- Automatic expiration (TTL)
- No complex queries needed
- Horizontal scaling

**Recommended:** Redis, Memcached

### 4.2 Caching Layer

```python
import redis
import json
import hashlib

r = redis.Redis()

def get_product_cached(product_id):
    cache_key = f"product:{product_id}"

    # Try cache first
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)

    # Cache miss - fetch from database
    product = database.query(f"SELECT * FROM products WHERE id = {product_id}")

    # Store in cache with 5 minute TTL
    r.setex(cache_key, 300, json.dumps(product))
    return product

def invalidate_product_cache(product_id):
    r.delete(f"product:{product_id}")

# Cache complex query results
def get_homepage_products():
    cache_key = "homepage:featured_products"
    cached = r.get(cache_key)
    if cached:
        return json.loads(cached)

    # Expensive query
    products = database.query("""
        SELECT p.*, AVG(r.rating) as avg_rating
        FROM products p
        JOIN reviews r ON p.id = r.product_id
        GROUP BY p.id
        ORDER BY avg_rating DESC
        LIMIT 20
    """)

    r.setex(cache_key, 600, json.dumps(products))  # 10 min cache
    return products
```

**Recommended:** Redis, Memcached

### 4.3 Rate Limiting

```python
import redis
import time

r = redis.Redis()

def rate_limit(user_id, limit=100, window=60):
    """
    Sliding window rate limiter
    Returns True if request allowed, False if rate limited
    """
    key = f"rate_limit:{user_id}"
    now = time.time()
    window_start = now - window

    pipe = r.pipeline()

    # Remove old entries
    pipe.zremrangebyscore(key, 0, window_start)

    # Count requests in window
    pipe.zcard(key)

    # Add current request
    pipe.zadd(key, {str(now): now})

    # Set expiry
    pipe.expire(key, window)

    results = pipe.execute()
    request_count = results[1]

    return request_count < limit

# Usage
if rate_limit("user123", limit=100, window=60):
    process_request()
else:
    return "Rate limited", 429
```

**Recommended:** Redis

### 4.4 Leaderboards and Rankings

```python
import redis

r = redis.Redis()

# Add/update score
def update_score(game_id, user_id, score):
    r.zadd(f"leaderboard:{game_id}", {user_id: score})

# Get top 10
def get_top_players(game_id, count=10):
    return r.zrevrange(f"leaderboard:{game_id}", 0, count-1, withscores=True)

# Get user rank
def get_user_rank(game_id, user_id):
    rank = r.zrevrank(f"leaderboard:{game_id}", user_id)
    return rank + 1 if rank is not None else None

# Get users around a specific user
def get_nearby_players(game_id, user_id, count=5):
    rank = r.zrevrank(f"leaderboard:{game_id}", user_id)
    if rank is None:
        return []
    start = max(0, rank - count)
    end = rank + count
    return r.zrevrange(f"leaderboard:{game_id}", start, end, withscores=True)
```

**Recommended:** Redis (sorted sets are perfect for this)

---

## 5. Wide-Column Store Use Cases

### 5.1 Time-Series Data (IoT Sensors)

```cql
-- Cassandra schema for IoT data
CREATE KEYSPACE iot WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3
};

CREATE TABLE sensor_readings (
    sensor_id UUID,
    date DATE,
    timestamp TIMESTAMP,
    temperature FLOAT,
    humidity FLOAT,
    pressure FLOAT,
    PRIMARY KEY ((sensor_id, date), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Write sensor data (optimized for writes)
INSERT INTO sensor_readings
    (sensor_id, date, timestamp, temperature, humidity, pressure)
VALUES
    (uuid(), '2024-01-15', toTimestamp(now()), 23.5, 45.2, 1013.25);

-- Query last 24 hours for a sensor
SELECT * FROM sensor_readings
WHERE sensor_id = ? AND date = '2024-01-15'
AND timestamp > '2024-01-14 00:00:00';
```

**Why Wide-Column:**
- Handles millions of writes per second
- Natural time-series partitioning
- Automatic data distribution
- Tunable consistency

**Recommended:** Cassandra, ScyllaDB, HBase

### 5.2 User Activity Tracking

```cql
-- Activity feed schema
CREATE TABLE user_activity (
    user_id UUID,
    activity_date DATE,
    activity_time TIMESTAMP,
    activity_type TEXT,
    metadata MAP<TEXT, TEXT>,
    PRIMARY KEY ((user_id, activity_date), activity_time)
) WITH CLUSTERING ORDER BY (activity_time DESC);

-- Write activity
INSERT INTO user_activity
    (user_id, activity_date, activity_time, activity_type, metadata)
VALUES (
    uuid(),
    '2024-01-15',
    toTimestamp(now()),
    'page_view',
    {'page': '/products/123', 'referrer': 'google.com', 'duration': '45s'}
);

-- Get user's today activity
SELECT * FROM user_activity
WHERE user_id = ? AND activity_date = '2024-01-15'
LIMIT 100;
```

**Recommended:** Cassandra, ScyllaDB

### 5.3 Messaging and Chat Systems

```cql
-- Chat messages by conversation
CREATE TABLE messages (
    conversation_id UUID,
    message_id TIMEUUID,
    sender_id UUID,
    content TEXT,
    message_type TEXT,
    attachments LIST<TEXT>,
    read_by SET<UUID>,
    PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

-- Get recent messages
SELECT * FROM messages
WHERE conversation_id = ?
LIMIT 50;

-- Messages by user (for search)
CREATE TABLE messages_by_user (
    user_id UUID,
    sent_date DATE,
    message_id TIMEUUID,
    conversation_id UUID,
    content TEXT,
    PRIMARY KEY ((user_id, sent_date), message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

**Recommended:** Cassandra, ScyllaDB

---

## 6. Graph Database Use Cases

### 6.1 Social Network

```cypher
// Neo4j - Create social graph
CREATE (alice:Person {name: 'Alice', age: 30})
CREATE (bob:Person {name: 'Bob', age: 28})
CREATE (charlie:Person {name: 'Charlie', age: 35})

CREATE (alice)-[:FOLLOWS]->(bob)
CREATE (bob)-[:FOLLOWS]->(charlie)
CREATE (alice)-[:FOLLOWS]->(charlie)

// Friend recommendations (friends of friends)
MATCH (user:Person {name: 'Alice'})-[:FOLLOWS]->(:Person)-[:FOLLOWS]->(recommended:Person)
WHERE NOT (user)-[:FOLLOWS]->(recommended) AND user <> recommended
RETURN recommended.name, COUNT(*) as mutual_friends
ORDER BY mutual_friends DESC
LIMIT 5;

// Shortest path between users
MATCH path = shortestPath(
    (alice:Person {name: 'Alice'})-[:FOLLOWS*]-(bob:Person {name: 'Bob'})
)
RETURN path;

// Influence score (PageRank-like)
MATCH (p:Person)
RETURN p.name, size((p)<-[:FOLLOWS]-()) as followers
ORDER BY followers DESC;
```

**Why Graph DB:**
- Natural representation of relationships
- Efficient traversal queries
- Friend recommendations in milliseconds
- Complex relationship patterns

**Recommended:** Neo4j, Amazon Neptune

### 6.2 Fraud Detection

```cypher
// Detect suspicious transaction patterns
// Find accounts connected through multiple shared attributes

MATCH (a1:Account)-[:HAS_PHONE]->(phone:Phone)<-[:HAS_PHONE]-(a2:Account)
WHERE a1 <> a2
WITH a1, a2, COUNT(phone) as shared_phones

MATCH (a1)-[:HAS_EMAIL]->(email:Email)<-[:HAS_EMAIL]-(a2)
WITH a1, a2, shared_phones, COUNT(email) as shared_emails

MATCH (a1)-[:HAS_DEVICE]->(device:Device)<-[:HAS_DEVICE]-(a2)
WITH a1, a2, shared_phones, shared_emails, COUNT(device) as shared_devices

WHERE shared_phones > 0 OR shared_emails > 0 OR shared_devices > 0
RETURN a1.id, a2.id, shared_phones, shared_emails, shared_devices
ORDER BY shared_phones + shared_emails + shared_devices DESC;

// Transaction chain analysis
MATCH path = (source:Account)-[:TRANSFERRED_TO*1..5]->(destination:Account)
WHERE source.flagged = true
RETURN path;
```

**Recommended:** Neo4j, TigerGraph

### 6.3 Knowledge Graph / Recommendation Engine

```cypher
// Product recommendations based on purchase patterns
MATCH (user:Customer {id: 'cust123'})-[:PURCHASED]->(product:Product)
      <-[:PURCHASED]-(other:Customer)-[:PURCHASED]->(recommended:Product)
WHERE NOT (user)-[:PURCHASED]->(recommended)
RETURN recommended.name, COUNT(DISTINCT other) as score
ORDER BY score DESC
LIMIT 10;

// Content-based recommendations
MATCH (user:Customer {id: 'cust123'})-[:PURCHASED]->(p:Product)-[:IN_CATEGORY]->(c:Category)
MATCH (recommended:Product)-[:IN_CATEGORY]->(c)
WHERE NOT (user)-[:PURCHASED]->(recommended)
RETURN recommended.name, COUNT(c) as category_matches
ORDER BY category_matches DESC
LIMIT 10;
```

**Recommended:** Neo4j, Amazon Neptune

---

## 7. Search Database Use Cases

### 7.1 E-Commerce Search

```json
// Elasticsearch - Product search with facets
PUT /products/_doc/1
{
    "name": "Premium Wireless Headphones",
    "description": "High-fidelity audio with active noise cancellation",
    "brand": "AudioTech",
    "price": 299.99,
    "category": ["Electronics", "Audio", "Headphones"],
    "attributes": {
        "color": "black",
        "wireless": true,
        "battery_life": "30 hours"
    },
    "ratings": {
        "average": 4.5,
        "count": 1247
    }
}

// Search with filters and aggregations
GET /products/_search
{
    "query": {
        "bool": {
            "must": [
                { "multi_match": {
                    "query": "wireless headphones",
                    "fields": ["name^3", "description", "category"]
                }}
            ],
            "filter": [
                { "range": { "price": { "lte": 300 }}},
                { "term": { "attributes.wireless": true }}
            ]
        }
    },
    "aggs": {
        "brands": { "terms": { "field": "brand.keyword" }},
        "price_ranges": {
            "range": {
                "field": "price",
                "ranges": [
                    { "to": 100 },
                    { "from": 100, "to": 200 },
                    { "from": 200 }
                ]
            }
        }
    }
}
```

**Recommended:** Elasticsearch, Meilisearch

### 7.2 Log Analytics

```json
// Log ingestion
POST /logs/_doc
{
    "@timestamp": "2024-01-15T10:30:00Z",
    "level": "ERROR",
    "service": "api-gateway",
    "message": "Connection timeout to database",
    "host": "prod-server-01",
    "trace_id": "abc123",
    "response_time_ms": 5000,
    "metadata": {
        "database": "primary-postgres",
        "retry_count": 3
    }
}

// Error analysis query
GET /logs/_search
{
    "query": {
        "bool": {
            "must": [
                { "match": { "level": "ERROR" }},
                { "range": { "@timestamp": { "gte": "now-1h" }}}
            ]
        }
    },
    "aggs": {
        "errors_by_service": {
            "terms": { "field": "service.keyword" }
        },
        "errors_over_time": {
            "date_histogram": {
                "field": "@timestamp",
                "calendar_interval": "5m"
            }
        }
    }
}
```

**Recommended:** Elasticsearch (ELK Stack), Loki

---

## 8. Time-Series Database Use Cases

### 8.1 Application Metrics

```sql
-- TimescaleDB - Application performance metrics
CREATE TABLE metrics (
    time TIMESTAMPTZ NOT NULL,
    service VARCHAR(100),
    metric_name VARCHAR(100),
    value DOUBLE PRECISION,
    tags JSONB
);

SELECT create_hypertable('metrics', 'time');

-- Insert metrics
INSERT INTO metrics (time, service, metric_name, value, tags)
VALUES
    (NOW(), 'api-server', 'request_latency_ms', 45.2,
     '{"endpoint": "/api/users", "method": "GET"}');

-- Query: Average latency by endpoint (last hour)
SELECT
    time_bucket('5 minutes', time) AS bucket,
    tags->>'endpoint' as endpoint,
    AVG(value) as avg_latency,
    MAX(value) as max_latency,
    COUNT(*) as request_count
FROM metrics
WHERE
    metric_name = 'request_latency_ms'
    AND time > NOW() - INTERVAL '1 hour'
GROUP BY bucket, endpoint
ORDER BY bucket DESC;
```

**Recommended:** TimescaleDB, InfluxDB, Prometheus

### 8.2 Financial Market Data

```sql
-- Stock price time-series
CREATE TABLE stock_prices (
    time TIMESTAMPTZ NOT NULL,
    symbol VARCHAR(10),
    open DECIMAL(10,2),
    high DECIMAL(10,2),
    low DECIMAL(10,2),
    close DECIMAL(10,2),
    volume BIGINT
);

SELECT create_hypertable('stock_prices', 'time');

-- Moving averages
SELECT
    time_bucket('1 day', time) AS day,
    symbol,
    LAST(close, time) as closing_price,
    AVG(close) OVER (
        PARTITION BY symbol
        ORDER BY time_bucket('1 day', time)
        ROWS BETWEEN 19 PRECEDING AND CURRENT ROW
    ) as ma_20
FROM stock_prices
WHERE symbol = 'AAPL'
  AND time > NOW() - INTERVAL '3 months'
ORDER BY day;
```

**Recommended:** TimescaleDB, QuestDB, InfluxDB

---

## 9. Decision Matrix

| Use Case | Primary DB | Cache Layer | Search | Analytics |
|----------|-----------|-------------|--------|-----------|
| E-Commerce | PostgreSQL | Redis | Elasticsearch | ClickHouse |
| Social Network | PostgreSQL | Redis | Elasticsearch | Neo4j |
| IoT Platform | Cassandra | Redis | - | TimescaleDB |
| Gaming | MongoDB | Redis | - | ClickHouse |
| Banking | Oracle/PostgreSQL | - | - | Oracle |
| CMS | PostgreSQL | Redis | Elasticsearch | - |
| Chat/Messaging | Cassandra | Redis | Elasticsearch | - |
| Log Analytics | - | - | Elasticsearch | ClickHouse |

---

## 10. Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│                     KEY TAKEAWAYS                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  • No single database fits all use cases                            │
│  • Start with requirements, not technology                          │
│  • Consider polyglot persistence for complex systems                │
│  • Operational complexity matters (team skills, maintenance)        │
│  • Plan for scale, but don't over-engineer                         │
│  • Caching is often more impactful than database choice            │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

The best database is the one that:
1. Fits your data model naturally
2. Handles your query patterns efficiently
3. Scales to your needs
4. Your team can operate confidently
