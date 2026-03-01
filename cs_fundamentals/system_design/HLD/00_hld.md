# High Level Design (HLD)

High Level Design focuses on the architecture and components of distributed systems. HLD interviews typically last 45-60 minutes and assess your ability to design scalable, reliable systems.

---

## What is HLD?

HLD involves:
- **System architecture**: Components and their interactions
- **Scalability**: Handling millions of users
- **Reliability**: Fault tolerance and high availability
- **Trade-offs**: Consistency vs availability, latency vs throughput

---

## HLD Interview Structure

```
┌────────────────────────────────────────────────────────────────┐
│  1. Requirements (5 min)                                       │
│     - Clarify functional requirements                          │
│     - Identify non-functional requirements (scale, latency)    │
│     - Define scope (what's in/out)                             │
├────────────────────────────────────────────────────────────────┤
│  2. Estimation (5 min)                                         │
│     - Users, requests per second                               │
│     - Storage, bandwidth                                       │
│     - Read/write ratio                                         │
├────────────────────────────────────────────────────────────────┤
│  3. High Level Design (15 min)                                 │
│     - Draw system components                                   │
│     - Define APIs                                              │
│     - Data flow                                                │
├────────────────────────────────────────────────────────────────┤
│  4. Deep Dive (15-20 min)                                      │
│     - Database schema                                          │
│     - Specific component details                               │
│     - Handle edge cases                                        │
├────────────────────────────────────────────────────────────────┤
│  5. Wrap-up (5 min)                                            │
│     - Bottlenecks and solutions                                │
│     - Monitoring and alerting                                  │
│     - Future improvements                                      │
└────────────────────────────────────────────────────────────────┘
```

---

## Topics in This Section

### [6. Core Components](Core_Components/00_core_components.md)
The building blocks of distributed systems.
- Load Balancers
- CDN
- Message Queues
- API Gateway
- Service Discovery

### [7. Architecture Patterns](Architecture_Patterns/00_architecture_patterns.md)
Common patterns for system architecture.
- Microservices
- Event-Driven Architecture
- CQRS
- Saga Pattern

### [8. Case Studies](Case_Studies/00_case_studies.md)
Practice problems with detailed solutions.
- Design URL Shortener
- Design Twitter
- Design WhatsApp
- Design YouTube
- And more...

### [9. Interview Framework](Interview_Framework/00_interview_framework.md)
How to approach HLD interviews.
- Requirements gathering
- Back-of-envelope estimation
- Communication strategies

---

## Key Skills for HLD

### 1. Component Selection
```
"For caching, I would use Redis because..."
"For the database, PostgreSQL makes sense here because..."
"We need a message queue for async processing..."
```

### 2. Trade-off Analysis
```
"We could use strong consistency, but that would increase latency..."
"Sharding gives us scalability, but cross-shard queries become expensive..."
"Synchronous replication ensures no data loss, but reduces availability..."
```

### 3. Estimation
```
"With 100M DAU, assuming 10 requests/user/day = 1B requests/day"
"1B requests / 100K seconds ≈ 10K QPS"
"At peak (2x average) = 20K QPS"
```

### 4. Failure Handling
```
"If the primary database fails, we promote a replica..."
"With circuit breaker, we fail fast and prevent cascade failures..."
"Data is replicated across 3 regions for disaster recovery..."
```

---

## Common HLD Mistakes

1. **Jumping to solution too quickly**: Spend time on requirements
2. **Ignoring scale**: Always discuss scale and bottlenecks
3. **No trade-off discussion**: Every decision has pros/cons
4. **Monologue**: Make it a conversation with the interviewer
5. **Too much detail on one component**: Cover the full system first
6. **Ignoring non-functional requirements**: Latency, availability, security

---

## Quick Reference: Common Numbers

```
Read latency:
- L1 cache: 0.5 ns
- L2 cache: 7 ns
- RAM: 100 ns
- SSD: 150 μs
- HDD: 10 ms
- Network (same DC): 0.5 ms
- Network (cross-continent): 150 ms

Throughput:
- SSD sequential read: 500 MB/s
- HDD sequential read: 100 MB/s
- 1 Gbps network: 125 MB/s
- Redis: 100K+ ops/sec
- PostgreSQL: 10K-50K queries/sec

Storage:
- 1 char = 1-4 bytes (UTF-8)
- UUID = 36 chars = 36 bytes
- Timestamp = 8 bytes
- 1 Million = 10^6
- 1 Billion = 10^9
```
