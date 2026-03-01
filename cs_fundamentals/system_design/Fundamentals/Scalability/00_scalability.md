# Scalability

Scalability is the ability of a system to handle growing amounts of work by adding resources. It's one of the most critical concepts in system design interviews.

---

## Core Concepts

### Types of Scaling

| Type | Description | Pros | Cons |
|------|-------------|------|------|
| **Vertical (Scale Up)** | Add more power to existing machine | Simple, no code changes | Hardware limits, single point of failure |
| **Horizontal (Scale Out)** | Add more machines | No hardware limits, fault tolerant | Complex, requires load balancing |

### Key Metrics

- **Throughput**: Requests per second (RPS) the system can handle
- **Latency**: Time to process a single request (p50, p95, p99)
- **Availability**: Percentage of time the system is operational (99.9% = 8.76 hours downtime/year)

---

## Topics in This Section

- [1.1 Horizontal vs Vertical Scaling](01_horizontal_vs_vertical_scaling.md)
- [1.2 Load Balancing](02_load_balancing.md)
- [1.3 Database Replication](03_database_replication.md)

---

## Common Interview Questions

1. "How would you scale this system to 10x users?"
2. "What's your strategy for handling traffic spikes?"
3. "How do you ensure high availability?"

---

## Quick Reference

```
Users → Load Balancer → [App Server 1, App Server 2, ... App Server N]
                                    ↓
                    [Read Replica 1, Read Replica 2] ← Primary DB
                                    ↓
                            [Cache Cluster]
```
