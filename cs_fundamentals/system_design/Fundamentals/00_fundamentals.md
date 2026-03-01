# System Design Fundamentals

This section covers the foundational concepts required for system design interviews. Understanding these building blocks is essential before diving into specific system designs.

---

## Overview

Before designing any large-scale system, you must understand:
1. **Scalability** - How systems grow to handle more users/data
2. **CAP Theorem** - Trade-offs in distributed systems
3. **Databases** - Storage, indexing, and partitioning strategies
4. **Caching** - Reducing latency and database load
5. **Networking** - Communication protocols and patterns

---

## Learning Path

### [1. Scalability](Scalability/00_scalability.md)
Understanding how to scale systems from thousands to millions of users.
- Horizontal vs Vertical Scaling
- Load Balancing strategies
- Database Replication

### [2. CAP Theorem & Distributed Systems](CAP_Theorem/00_cap_theorem.md)
The fundamental trade-offs in distributed computing.
- Consistency vs Availability
- Eventual Consistency patterns
- Consensus algorithms (Paxos, Raft)

### [3. Database Fundamentals](Database_Fundamentals/00_database_fundamentals.md)
Choosing and optimizing the right database for your use case.
- SQL vs NoSQL trade-offs
- Indexing strategies
- Sharding and Partitioning

### [4. Caching](Caching/00_caching.md)
Reducing latency and improving performance through caching.
- Cache-aside, Write-through, Write-behind patterns
- Eviction policies (LRU, LFU, FIFO)
- Distributed caching (Redis, Memcached)

### [5. Networking Basics](Networking_Basics/00_networking_basics.md)
Communication protocols for modern distributed systems.
- HTTP/HTTPS and REST APIs
- WebSockets and real-time communication
- gRPC and Protocol Buffers

---

## Key Interview Points

When discussing fundamentals in interviews:
1. Always **quantify** - use numbers (QPS, latency, storage)
2. Discuss **trade-offs** - there's no perfect solution
3. Consider **failure modes** - what happens when X fails?
4. Think about **evolution** - how does this scale 10x?
