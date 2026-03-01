# HLD Case Studies

This section contains detailed system design case studies for common interview questions.

---

## Case Study List

### Foundational (Start Here)
- [8.1 Design URL Shortener (TinyURL)](01_design_url_shortener.md) - Simple, covers core concepts
- [8.2 Design Rate Limiter](02_design_rate_limiter.md) - Algorithm focused

### Social Media
- [8.3 Design Twitter/X](03_design_twitter.md) - Feed, posts, followers
- [8.4 Design Instagram](04_design_instagram.md) - Media-heavy

### Messaging
- [8.5 Design WhatsApp/Messenger](05_design_whatsapp.md) - Real-time chat

### Media
- [8.6 Design YouTube/Netflix](06_design_youtube.md) - Video streaming

### Transportation
- [8.7 Design Uber/Lyft](07_design_uber.md) - Location-based

### Infrastructure
- [8.8 Design Notification System](08_design_notification_system.md)
- [8.9 Design Search Autocomplete](09_design_search_autocomplete.md)
- [8.10 Design Distributed Cache](10_design_distributed_cache.md)
- [8.11 Design Web Crawler](11_design_web_crawler.md)

### E-Commerce
- [8.12 Design Ticket Booking System](12_design_ticket_booking.md)

---

## How to Use These Case Studies

### Study Approach
1. **Read requirements** - Understand the problem
2. **Try it yourself** - Spend 30 min designing
3. **Compare** - Check against the solution
4. **Identify gaps** - Note what you missed
5. **Practice again** - Repeat after a few days

### Interview Approach
1. **Clarify requirements** (5 min)
2. **Estimate scale** (5 min)
3. **High-level design** (15 min)
4. **Deep dive** (15-20 min)
5. **Wrap up** (5 min)

---

## Common Components Across Designs

Most system designs use these building blocks:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Client Apps                                 │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                       CDN (Static Assets)                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                          Load Balancer                                   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                           API Gateway                                    │
│              (Auth, Rate Limiting, Routing)                              │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
            │  Service A  │ │  Service B  │ │  Service C  │
            └─────────────┘ └─────────────┘ └─────────────┘
                    │               │               │
                    └───────────────┼───────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
            │    Cache    │ │  Database   │ │Message Queue│
            └─────────────┘ └─────────────┘ └─────────────┘
```

---

## Estimation Quick Reference

### Users and Traffic
```
Daily Active Users (DAU) = 100M
Each user: 10 reads, 1 write per day

Reads: 100M × 10 = 1B/day
Writes: 100M × 1 = 100M/day

Reads per second: 1B / 86400 ≈ 12K QPS
Writes per second: 100M / 86400 ≈ 1.2K QPS

Peak (2x average): 24K read QPS, 2.4K write QPS
```

### Storage
```
100M users × 1 KB profile = 100 GB
100M posts/day × 500 bytes = 50 GB/day = 18 TB/year
```

### Bandwidth
```
1B reads/day × 1 KB = 1 TB/day outgoing
100M writes/day × 500 bytes = 50 GB/day incoming
```

---

## Template for Answering

```markdown
## 1. Requirements
### Functional
- What features?

### Non-Functional
- Scale: How many users?
- Performance: Latency requirements?
- Availability: How many 9s?

## 2. Estimation
- QPS (queries per second)
- Storage
- Bandwidth

## 3. High Level Design
- Component diagram
- Data flow

## 4. API Design
- Key endpoints

## 5. Data Model
- Database schema
- Choice of database

## 6. Deep Dive
- Specific component details
- Trade-offs

## 7. Bottlenecks & Solutions
- What could fail?
- How to scale?
```
