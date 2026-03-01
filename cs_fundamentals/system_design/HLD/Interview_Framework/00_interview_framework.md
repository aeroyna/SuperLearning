# HLD Interview Framework

A structured approach to system design interviews.

---

## Topics in This Section

- [9.1 Requirements Gathering](01_requirements_gathering.md)
- [9.2 Back-of-Envelope Estimation](02_estimation_calculations.md)
- [9.3 High Level Design Steps](03_hld_steps.md)
- [9.4 Deep Dive Strategies](04_deep_dive_strategies.md)

---

## Interview Timeline (45-60 minutes)

```
┌─────────────────────────────────────────────────────────────────┐
│ 0-5 min    │ Requirements & Scope                               │
├─────────────┼────────────────────────────────────────────────────┤
│ 5-10 min   │ Estimation (scale, storage, bandwidth)             │
├─────────────┼────────────────────────────────────────────────────┤
│ 10-25 min  │ High Level Design (draw architecture)              │
├─────────────┼────────────────────────────────────────────────────┤
│ 25-45 min  │ Deep Dive (database, specific components)          │
├─────────────┼────────────────────────────────────────────────────┤
│ 45-50 min  │ Wrap up (bottlenecks, improvements, questions)     │
└─────────────┴────────────────────────────────────────────────────┘
```

---

## The RESHADED Framework

A mnemonic to ensure you cover all aspects:

```
R - Requirements (functional & non-functional)
E - Estimation (scale, traffic, storage)
S - Storage (database choice, schema)
H - High-level design (architecture diagram)
A - APIs (key endpoints)
D - Deep dive (detailed component design)
E - Edge cases (failure handling)
D - Discussion (trade-offs, improvements)
```

---

## Step-by-Step Approach

### 1. Requirements (5 min)
```
Interviewer: "Design Twitter"

You:
"Before I start, let me clarify some requirements.

Functional:
- Users can post tweets (text? images? videos?)
- Users can follow other users
- Users can see a feed of tweets from people they follow
- Users can like/retweet

Non-functional:
- How many users? (100M DAU?)
- What latency is acceptable for feed loading? (<500ms?)
- Should prioritize availability or consistency?

Scope:
- Should I focus on feed generation or the full system?
- Are we including DMs, notifications, search?"
```

### 2. Estimation (5 min)
```
"Let me do some quick calculations.

Users: 100M DAU
- Each user reads feed 10 times/day
- Each user posts 0.5 tweets/day

Traffic:
- Reads: 100M × 10 = 1B/day = 12K QPS
- Writes: 100M × 0.5 = 50M/day = 600 QPS

Storage:
- Tweet size: 300 bytes (text + metadata)
- 50M tweets/day = 15GB/day = 5TB/year

Does this scale sound right to you?"
```

### 3. High Level Design (15 min)
```
"Let me draw the architecture.

[Draw components]
- Clients
- CDN for static assets
- Load balancer
- API Gateway
- Tweet Service
- User Service
- Feed Service
- Timeline Cache
- Database(s)
- Message Queue for async processing

[Explain data flow]
- Write path: How tweets are created
- Read path: How feed is generated"
```

### 4. API Design
```
"Here are the key APIs:

POST /tweets
- Body: { content, media_ids }
- Returns: { tweet_id, created_at }

GET /feed?cursor=xxx&limit=20
- Returns: { tweets: [...], next_cursor }

POST /users/{id}/follow
DELETE /users/{id}/follow
```

### 5. Deep Dive (15-20 min)
```
Focus on 1-2 components in detail:

"Let's dive into feed generation.

Option 1: Pull model
- Query followers at read time
- Join with their tweets
- Sort and return

Option 2: Push model (Fan-out on write)
- When user tweets, push to followers' feed caches
- Read is fast (just fetch from cache)

Option 3: Hybrid
- Push for normal users
- Pull for celebrities (many followers)

I recommend the hybrid approach because..."
```

### 6. Wrap Up (5 min)
```
"Let me discuss potential bottlenecks:

1. Hot users (celebrities)
   - Solution: Hybrid fan-out model

2. Feed cache size
   - Solution: Store only recent tweets, LRU eviction

3. Database scaling
   - Solution: Shard by user_id

Future improvements:
- ML-based feed ranking
- Real-time notifications
- Search functionality"
```

---

## Common Mistakes to Avoid

| Mistake | How to Avoid |
|---------|--------------|
| Jumping to solution | Spend time on requirements first |
| Monologue | Check in with interviewer regularly |
| Too much detail on one area | Cover breadth before depth |
| Ignoring scale | Always discuss numbers |
| No trade-offs | Every decision has pros/cons |
| Ignoring failure | What if X goes down? |

---

## Communication Tips

### Do
- "Let me clarify the requirements..."
- "I'm making an assumption that X..."
- "The trade-off here is..."
- "Let me check if this makes sense before continuing..."
- "Should I dive deeper into X or move on to Y?"

### Don't
- Jump into coding/details immediately
- Stay silent for long periods
- Ignore hints from the interviewer
- Argue about the "right" solution
- Say "I don't know" without trying

---

## Handling Curve Balls

### "What if we need to support 10x more users?"
- Discuss horizontal scaling
- Sharding strategies
- Caching layers
- CDN optimization

### "How would you handle this failure?"
- Replication for database
- Circuit breakers for services
- Graceful degradation
- Retry with backoff

### "Can you be more specific about X?"
- Draw detailed component diagram
- Walk through data flow
- Discuss specific algorithms
- Show database schema

---

## Practice Checklist

Before the interview, practice:
- [ ] Drawing architecture diagrams
- [ ] Back-of-envelope calculations
- [ ] Explaining trade-offs clearly
- [ ] Database schema design
- [ ] API design
- [ ] Discussing failure modes
- [ ] Time management (don't exceed 45-50 min per section)
