# HLD Interview Template

Use this template as a mental checklist during HLD interviews.

---

## Phase 1: Requirements (5 minutes)

### Questions to Ask

```markdown
**Functional Requirements**
- What are the core features?
- Who are the users?
- What actions can users perform?

**Non-Functional Requirements**
- How many users? (DAU, MAU)
- What latency is acceptable?
- Availability requirements?
- Consistency requirements?

**Scope**
- What should I focus on?
- What's out of scope?
```

### Output
```
Functional:
- Users can [action 1]
- Users can [action 2]
- System should [capability]

Non-Functional:
- 100M DAU
- < 200ms latency for reads
- 99.9% availability
- Eventual consistency acceptable

Scope:
- Focus on: [area]
- Out of scope: [excluded features]
```

---

## Phase 2: Estimation (5 minutes)

### Traffic
```
DAU × actions/user = daily requests
daily requests / 100,000 = QPS
QPS × 2-3 = peak QPS
```

### Storage
```
records/day × record_size = daily storage
daily storage × 365 × years = total storage
```

### Bandwidth
```
QPS × response_size = bandwidth
```

### Output
```
Traffic:
- 100M DAU × 10 actions = 1B requests/day
- 1B / 100K = 10K QPS, peak 30K QPS

Storage:
- 50M records/day × 500 bytes = 25 GB/day
- 25 GB × 365 × 5 = 45 TB over 5 years

Bandwidth:
- 10K QPS × 1KB = 10 MB/s
```

---

## Phase 3: High Level Design (15 minutes)

### Draw Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                          Clients                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                       CDN / DNS                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      Load Balancer                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                       API Gateway                            │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
        ┌───────────┐   ┌───────────┐   ┌───────────┐
        │ Service A │   │ Service B │   │ Service C │
        └───────────┘   └───────────┘   └───────────┘
              │               │               │
              ↓               ↓               ↓
        ┌───────────┐   ┌───────────┐   ┌───────────┐
        │   Cache   │   │  Database │   │   Queue   │
        └───────────┘   └───────────┘   └───────────┘
```

### Define APIs
```
POST /api/v1/[resource]
  Body: { ... }
  Response: { id, ... }

GET /api/v1/[resource]/{id}
  Response: { ... }

GET /api/v1/[resource]?cursor=X&limit=20
  Response: { data: [...], next_cursor }
```

### Describe Data Flow
```
Write Path:
1. Client sends request to API Gateway
2. Gateway routes to [Service]
3. Service writes to [Database]
4. Publishes event to [Queue]
5. Returns response to client

Read Path:
1. Client sends request
2. Check [Cache]
3. If miss, query [Database]
4. Cache result
5. Return to client
```

---

## Phase 4: Deep Dive (15-20 minutes)

### Database Design
```sql
CREATE TABLE [entity] (
    id UUID PRIMARY KEY,
    [field1] TYPE,
    [field2] TYPE,
    created_at TIMESTAMP,
    INDEX idx_[field] ([field])
);
```

### Scaling Considerations
```
Read scaling:
- Add replicas
- Cache layer

Write scaling:
- Sharding by [key]
- Async processing

Database choice:
- SQL for [X] because [reason]
- NoSQL for [Y] because [reason]
```

### Component Deep Dive
Pick 1-2 components to explain in detail:
- How does caching work?
- How do we handle failures?
- What's the sharding strategy?

---

## Phase 5: Wrap Up (5 minutes)

### Bottlenecks & Solutions
```
1. Database becomes bottleneck
   → Shard by [key], add read replicas

2. Hot spots
   → Cache hot data, rate limiting

3. Single point of failure
   → Replicate across regions
```

### Future Improvements
```
- Add ML-based recommendations
- Support international users
- Real-time analytics
```

### Questions for Interviewer
```
- Any specific component you'd like me to elaborate on?
- Did I miss any important requirements?
```

---

## Checklist

```
[ ] Clarified requirements
[ ] Estimated scale (QPS, storage, bandwidth)
[ ] Drew architecture diagram
[ ] Defined key APIs
[ ] Described data flow
[ ] Designed database schema
[ ] Discussed scaling strategy
[ ] Addressed bottlenecks
[ ] Mentioned monitoring/alerting
```
