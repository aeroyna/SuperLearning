# HLD Interview Steps

A structured approach to system design interviews for High-Level Design questions.

## Overview

The HLD interview typically lasts 45-60 minutes. This guide provides a framework to structure your approach and ensure you cover all important aspects.

## Step 1: Clarify Requirements (3-5 minutes)

### Functional Requirements

Ask questions to understand what the system should do:

```
"Before I start designing, I'd like to clarify the requirements..."

1. Core Features
   - "What are the primary use cases?"
   - "Who are the users of this system?"
   - "What actions can users perform?"

2. Feature Scope
   - "Should I focus on [feature X] or is [feature Y] also in scope?"
   - "Are there any features we should explicitly NOT consider?"

3. User Flows
   - "Can you walk me through the main user journey?"
   - "What happens when [specific scenario]?"
```

### Non-Functional Requirements

Understand quality attributes:

```
1. Scale
   - "How many users are we expecting?"
   - "What's the read/write ratio?"
   - "What's the expected growth rate?"

2. Performance
   - "What latency is acceptable?"
   - "Are there any SLAs we need to meet?"

3. Availability
   - "What's the uptime requirement? 99.9%? 99.99%?"
   - "Is this a global system or regional?"

4. Consistency
   - "Is eventual consistency acceptable?"
   - "What data must be strongly consistent?"
```

### Example: URL Shortener

```
Functional:
- Create short URLs from long URLs
- Redirect short URLs to original
- Custom aliases? Analytics? Expiration?

Non-Functional:
- 100M URLs created/month
- 10:1 read/write ratio
- Low latency redirects (<100ms)
- 99.9% availability
```

## Step 2: Capacity Estimation (3-5 minutes)

### Traffic Estimation

Calculate requests per second:

```python
# Example: Twitter-like system

# Users
total_users = 500_000_000
daily_active_users = 200_000_000  # 40% DAU

# Read operations
tweets_read_per_day = 200_000_000 * 100  # 100 tweets per user
read_qps = 20_000_000_000 / 86400  # ~230K QPS

# Write operations
tweets_per_day = 200_000_000 * 0.5  # 0.5 tweets per user
write_qps = 100_000_000 / 86400  # ~1.2K QPS

# Peak traffic (2-3x average)
peak_read_qps = 230_000 * 3  # ~700K QPS
peak_write_qps = 1_200 * 3   # ~3.6K QPS
```

### Storage Estimation

Calculate storage needs:

```python
# Storage per tweet
tweet_text = 280  # bytes
metadata = 200    # bytes (user_id, timestamp, etc.)
media_reference = 100  # bytes
total_per_tweet = 580  # bytes

# Daily storage
new_tweets_per_day = 100_000_000
daily_storage = 100_000_000 * 580  # ~58 GB/day

# Media storage (10% have images)
images_per_day = 10_000_000
image_size = 500_000  # 500KB average
daily_media = 10_000_000 * 500_000  # 5 TB/day

# 5-year projection
total_storage = (58 * 365 * 5) + (5000 * 365 * 5)  # ~9 PB
```

### Bandwidth Estimation

```python
# Incoming (writes)
incoming_bandwidth = write_qps * 580  # bytes/second
# ~700 KB/s for tweets

# Outgoing (reads)
outgoing_bandwidth = read_qps * 580  # bytes/second
# ~130 MB/s for tweets

# With media
peak_outgoing = 700_000 * 500_000  # ~350 GB/s peak
```

### Quick Reference Formulas

| Metric | Formula |
|--------|---------|
| QPS | DAU × operations_per_user / 86400 |
| Peak QPS | Average QPS × 2-3 |
| Storage/day | operations_per_day × size_per_operation |
| Storage/year | Storage/day × 365 |
| Bandwidth | QPS × average_response_size |

## Step 3: High-Level Design (10-15 minutes)

### Start with Basic Architecture

Draw the fundamental components:

```
┌─────────┐     ┌─────────────┐     ┌──────────┐
│ Clients │────►│ Load        │────►│ Web      │
│         │     │ Balancer    │     │ Servers  │
└─────────┘     └─────────────┘     └────┬─────┘
                                         │
                                         ▼
                                   ┌──────────┐
                                   │ Database │
                                   └──────────┘
```

### Add Components Progressively

1. **API Layer**
   ```
   - Define main API endpoints
   - Consider REST vs GraphQL
   - Version the API
   ```

2. **Application Layer**
   ```
   - Stateless services
   - Clear separation of concerns
   - Microservices if needed
   ```

3. **Data Layer**
   ```
   - Primary database
   - Caching layer
   - CDN for static content
   ```

4. **Supporting Services**
   ```
   - Message queues
   - Background workers
   - Search/analytics
   ```

### Example: Complete Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CDN (Static Assets)                      │
└─────────────────────────────────────────────────────────────────┘
                                 │
┌─────────────────────────────────────────────────────────────────┐
│                    Load Balancer (L7)                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
┌───────────────────────────┴─────────────────────────────────────┐
│                    API Gateway                                   │
│            (Authentication, Rate Limiting, Routing)              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│ User Service  │   │ Post Service  │   │ Feed Service  │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌───────▼───────┐
                    │ Cache (Redis) │
                    └───────┬───────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼───────┐   ┌───────▼───────┐   ┌───────▼───────┐
│  User DB      │   │  Post DB      │   │  Graph DB     │
│  (MySQL)      │   │ (Cassandra)   │   │  (Neo4j)      │
└───────────────┘   └───────────────┘   └───────────────┘
```

## Step 4: Deep Dive Components (15-20 minutes)

### Choose 2-3 Critical Components

Based on the problem, identify what's most important:

| Problem Type | Focus Areas |
|-------------|-------------|
| Social Feed | Feed generation, fan-out, caching |
| Messaging | Real-time delivery, message ordering |
| Video Streaming | CDN, transcoding, adaptive bitrate |
| E-commerce | Inventory, transactions, consistency |
| Search | Indexing, ranking, autocomplete |

### Deep Dive Framework

For each component:

```
1. What problem does it solve?
2. How does it work internally?
3. What are the trade-offs?
4. How does it scale?
5. How does it handle failures?
```

### Example: Deep Dive on Caching

```
1. Problem: Reduce database load, improve latency

2. Design:
   - Cache-aside pattern
   - Redis cluster with 64GB nodes
   - LRU eviction policy
   - 15-minute TTL

3. Data Structure:
   user:{id}:profile → JSON blob
   user:{id}:feed → Sorted set of post IDs
   post:{id} → JSON blob

4. Scale:
   - Shard by user_id
   - Read replicas for hot data
   - Local L1 cache in app servers

5. Failure Handling:
   - Graceful degradation to DB
   - Cache stampede prevention
   - Circuit breaker pattern
```

## Step 5: Address Bottlenecks (5-10 minutes)

### Identify Potential Issues

```
1. Single Points of Failure
   - Database master
   - Load balancer
   - DNS

2. Performance Bottlenecks
   - Database reads/writes
   - Network latency
   - CPU-intensive operations

3. Scalability Limits
   - Vertical scaling limits
   - Data growth
   - Traffic spikes
```

### Propose Solutions

| Issue | Solution |
|-------|----------|
| DB single point | Primary-replica, multi-master |
| High read load | Caching, read replicas |
| High write load | Sharding, write-behind cache |
| Large data | Partitioning, archival |
| Traffic spikes | Auto-scaling, queue-based |
| Global latency | Multi-region, CDN |

### Scaling Strategies

```
Vertical Scaling:
- Bigger machines
- More memory
- Faster disks

Horizontal Scaling:
- More servers
- Sharding
- Replication

Functional Partitioning:
- Separate by feature
- Microservices
- Dedicated databases
```

## Step 6: Wrap Up (2-3 minutes)

### Summarize Key Decisions

```
"To summarize the design:

1. Architecture: Microservices with API Gateway
2. Data: Sharded MySQL for users, Cassandra for posts
3. Caching: Redis cluster for hot data
4. Scale: Can handle 1M QPS with current design
5. Trade-offs: Chose eventual consistency for availability"
```

### Discuss Future Improvements

```
"Given more time, I would also consider:
- ML-based ranking for feed
- A/B testing infrastructure
- Better observability
- Cost optimization"
```

## Common Mistakes to Avoid

### 1. Jumping to Solutions
❌ "Let's use Kafka..."
✅ "First, let me understand the requirements..."

### 2. Over-Engineering
❌ Adding microservices for a simple app
✅ Start simple, scale when needed

### 3. Ignoring Trade-offs
❌ "This solution is perfect"
✅ "This trades off X for Y"

### 4. Not Asking Questions
❌ Making assumptions silently
✅ Clarifying scope and constraints

### 5. Forgetting Non-Functional Requirements
❌ Only discussing features
✅ Addressing scale, reliability, performance

## Time Management

| Step | Time | Focus |
|------|------|-------|
| Requirements | 3-5 min | Clarify scope |
| Estimation | 3-5 min | Quick math |
| High-Level Design | 10-15 min | Draw architecture |
| Deep Dive | 15-20 min | 2-3 components |
| Bottlenecks | 5-10 min | Scale & reliability |
| Wrap Up | 2-3 min | Summarize |

## Related Topics

- [[01_approaching_interviews|Interview Approach]]
- [[02_estimation_techniques|Estimation Techniques]]
- [[04_deep_dive_strategies|Deep Dive Strategies]]
- [[../Case_Studies/00_case_studies|Case Studies]]

---

**Tags**: #system-design #hld #interview #framework
