# Mock Interview Scenarios

> Practice scenarios with realistic interviewer interactions, follow-up questions, and guidance on navigating ambiguity.

## Overview

These scenarios simulate actual Google system design interviews, including the back-and-forth dialogue, unexpected pivots, and deep-dive requests you'll encounter.

---

## Scenario 1: Design a Global Notification System

### Opening (Interviewer)

> "I'd like you to design a notification system for a large-scale application. Think about how users receive notifications across web, mobile push, email, and SMS."

### Candidate Response: Clarifying Questions

```
"Before I dive in, I'd like to understand the scope better:

1. What's our scale? How many users and notifications per day?"
   → "Assume 500M users, 5B notifications/day at peak"

2. "What notification types are most critical?"
   → "Push and in-app are most important, email is secondary"

3. "Are there real-time requirements?"
   → "Push should be within 1 second, email within minutes is fine"

4. "Do we need guaranteed delivery, or is best-effort okay?"
   → "At-least-once delivery. Users shouldn't miss critical notifications"

5. "Any preference for notification priority?"
   → "Yes, some are urgent (security alerts) vs informational (friend activity)"
```

### Requirements Summary

```
Functional:
- Multi-channel delivery (push, in-app, email, SMS)
- User preferences (opt-in/out per channel)
- Notification templates
- Scheduled notifications
- Delivery tracking

Non-Functional:
- 5B notifications/day (~58K/sec average, 200K/sec peak)
- Push latency < 1 second
- 99.9% delivery rate
- At-least-once delivery
```

### High-Level Design

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Global Notification System                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Event Sources                                                              │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                          │
│   │ Service │ │ Service │ │  Cron   │ │  Admin  │                          │
│   │    A    │ │    B    │ │  Jobs   │ │   UI    │                          │
│   └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘                          │
│        │           │           │           │                                │
│        └───────────┴─────┬─────┴───────────┘                                │
│                          ▼                                                   │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │                      Notification API                                 │ │
│   │  POST /notifications                                                  │ │
│   │  {user_ids: [...], template: "...", priority: "high", ...}           │ │
│   └──────────────────────────────┬───────────────────────────────────────┘ │
│                                  │                                          │
│                                  ▼                                          │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │                    Notification Service                               │ │
│   │                                                                       │ │
│   │  • Validate request             • Apply user preferences             │ │
│   │  • Render template              • Determine channels                 │ │
│   │  • Rate limiting                • Write to Pub/Sub                   │ │
│   └──────────────────────────────────┬───────────────────────────────────┘ │
│                                      │                                      │
│                                      ▼                                      │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │                      Pub/Sub (Priority Topics)                        │ │
│   │                                                                       │ │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │ │
│   │   │   urgent    │  │   normal    │  │    batch    │                  │ │
│   │   │   topic     │  │   topic     │  │    topic    │                  │ │
│   │   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                  │ │
│   └──────────┼────────────────┼────────────────┼─────────────────────────┘ │
│              │                │                │                            │
│              ▼                ▼                ▼                            │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                       Channel Workers                                │  │
│   │                                                                      │  │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐            │  │
│   │  │  Push    │  │ In-App   │  │  Email   │  │   SMS    │            │  │
│   │  │ Worker   │  │ Worker   │  │ Worker   │  │ Worker   │            │  │
│   │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘            │  │
│   └───────┼─────────────┼─────────────┼─────────────┼────────────────────┘  │
│           │             │             │             │                       │
│           ▼             ▼             ▼             ▼                       │
│   ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐              │
│   │   FCM /   │  │  WebSocket│  │ SendGrid/ │  │  Twilio   │              │
│   │   APNs    │  │  Servers  │  │   SES     │  │           │              │
│   └───────────┘  └───────────┘  └───────────┘  └───────────┘              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Interviewer Follow-up 1

> "How would you handle a viral event where notification volume spikes 100x?"

```
Candidate Response:

"Good question. For handling 100x spikes:

1. **Backpressure at ingestion**: The API layer uses adaptive rate
   limiting. When workers are overloaded, we reject or delay low-priority
   notifications rather than dropping high-priority ones.

2. **Priority queues**: Urgent notifications (security alerts) go to
   a separate topic with dedicated workers that are never starved.

3. **Auto-scaling**: Workers auto-scale based on queue depth.
   For Pub/Sub, Google handles the scaling automatically.

4. **Graceful degradation**:
   - If push delivery is slow, we don't retry immediately
   - Email can be batched (digests instead of individual emails)
   - In-app notifications can be retrieved on next app open

5. **Circuit breakers**: If FCM or APNs is slow, we circuit-break
   and queue for later rather than overwhelming them.

The key insight is that we sacrifice latency for low-priority
notifications to protect delivery of high-priority ones."
```

### Interviewer Follow-up 2

> "Let's dive deeper into the push notification delivery. Walk me through exactly what happens when we need to send a push to a user's phone."

```
Candidate Response:

"Let me trace the path from worker to user's device:

┌─────────────────────────────────────────────────────────────────┐
│                 Push Notification Deep Dive                      │
│                                                                  │
│   1. Push Worker receives message from Pub/Sub                   │
│      {user_id: 123, title: "New message", body: "..."}          │
│                                                                  │
│   2. Lookup user's device tokens                                 │
│      Query: device_tokens WHERE user_id = 123                    │
│      Result: [{token: "abc...", platform: "ios"},               │
│               {token: "xyz...", platform: "android"}]           │
│                                                                  │
│   3. Format for each platform                                    │
│      iOS: APNs payload format                                   │
│      Android: FCM payload format                                │
│                                                                  │
│   4. Send to platform providers                                  │
│      ┌─────────────┐       ┌─────────────┐                      │
│      │   APNs      │       │    FCM      │                      │
│      │  (Apple)    │       │  (Google)   │                      │
│      └──────┬──────┘       └──────┬──────┘                      │
│             │                     │                              │
│             ▼                     ▼                              │
│      ┌─────────────┐       ┌─────────────┐                      │
│      │   iPhone    │       │  Android    │                      │
│      └─────────────┘       └─────────────┘                      │
│                                                                  │
│   5. Handle responses                                            │
│      • Success: Mark as delivered                                │
│      • Invalid token: Remove from database                       │
│      • Retry-able error: Re-queue with backoff                  │
│      • Rate limited: Back off, retry later                      │
│                                                                  │
│   Key consideration: Token management                            │
│   - Tokens can become invalid when user reinstalls app          │
│   - Need to handle feedback from APNs/FCM                        │
│   - Store multiple tokens per user (multiple devices)           │
└─────────────────────────────────────────────────────────────────┘"
```

### Interviewer Follow-up 3

> "How do you ensure we don't send duplicate notifications?"

```
Candidate Response:

"Deduplication happens at multiple levels:

1. **Message-level deduplication**:
   - Each notification has a unique ID (hash of content + user + timestamp)
   - Before sending, check against a Redis cache:
     SETNX notification:{id} 1 EX 3600
   - If key exists, notification was already processed

2. **Idempotent processing**:
   - Workers use Pub/Sub's acknowledgment mechanism
   - Only ack after successful send to provider
   - If worker crashes mid-processing, message redelivers
   - The Redis check catches the duplicate on retry

3. **Provider-level deduplication**:
   - FCM supports collapse_key for grouping
   - Multiple notifications with same collapse_key = one displayed
   - Useful for 'you have N new messages' scenarios

4. **User-level suppression**:
   - Track recently sent notifications per user
   - Prevent 'notification storms' from chatty events
   - Example: rate limit to max 10 notifications/minute per user

The trade-off is between guaranteed delivery (at-least-once) and
perfect deduplication. We prefer occasional duplicates over missed
notifications, but minimize duplicates through these layers."
```

---

## Scenario 2: Design a Real-Time Collaborative Editor

### Opening (Interviewer)

> "Design a system like Google Docs where multiple users can edit the same document simultaneously."

### Clarifying Questions

```
"Let me understand the requirements:

1. How many concurrent editors per document?"
   → "Up to 100 simultaneous editors"

2. "What's the acceptable latency for seeing others' changes?"
   → "Under 100ms ideally, 500ms maximum"

3. "Do we need offline support?"
   → "Yes, users should be able to edit offline and sync later"

4. "What about document size?"
   → "Documents can be up to 10MB, mostly text with some images"

5. "History and versioning?"
   → "Yes, need to see revision history and restore previous versions"
```

### The Core Challenge: Concurrency

```
"The fundamental challenge here is handling concurrent edits without
conflicts. Let me explain the approaches:

┌─────────────────────────────────────────────────────────────────┐
│              Concurrency Approaches Comparison                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. LOCKING (Traditional)                                        │
│     User A locks paragraph → User B waits                        │
│     ❌ Poor UX, defeats real-time collaboration                 │
│                                                                  │
│  2. LAST-WRITE-WINS                                              │
│     Latest edit overwrites previous                              │
│     ❌ Users lose work                                           │
│                                                                  │
│  3. OPERATIONAL TRANSFORMATION (OT)                             │
│     Transform operations against concurrent edits                │
│     ✓ Google Docs uses this                                      │
│     ⚠️ Complex to implement correctly                            │
│                                                                  │
│  4. CONFLICT-FREE REPLICATED DATA TYPES (CRDTs)                 │
│     Data structures that merge automatically                     │
│     ✓ Simpler correctness guarantees                             │
│     ⚠️ Higher storage overhead                                   │
│                                                                  │
│  Recommendation: OT for text, similar to Google's approach      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘"
```

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  Real-Time Collaborative Editor                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                          Client Layer                                │  │
│   │                                                                      │  │
│   │  ┌────────────────────────────────────────────────────────────────┐ │  │
│   │  │                      Editor Client                              │ │  │
│   │  │                                                                 │ │  │
│   │  │  • Local document model (CRDT or OT state)                     │ │  │
│   │  │  • Pending operations queue                                     │ │  │
│   │  │  • WebSocket connection to collaboration server                 │ │  │
│   │  │  • Cursor/selection tracking                                    │ │  │
│   │  │  • Undo/redo stack                                              │ │  │
│   │  └────────────────────────────────────────────────────────────────┘ │  │
│   └───────────────────────────────┬─────────────────────────────────────┘  │
│                                   │ WebSocket                               │
│                                   ▼                                         │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                   Collaboration Service                              │  │
│   │                                                                      │  │
│   │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐ │  │
│   │  │  Session Manager│  │   OT Engine     │  │  Presence Service   │ │  │
│   │  │                 │  │                 │  │                     │ │  │
│   │  │  • Active users │  │  • Transform    │  │  • Who's editing    │ │  │
│   │  │  • Room routing │  │  • Compose      │  │  • Cursor positions │ │  │
│   │  │                 │  │  • Apply        │  │  • Typing indicators│ │  │
│   │  └────────┬────────┘  └────────┬────────┘  └──────────┬──────────┘ │  │
│   └───────────┼────────────────────┼─────────────────────┬┼────────────┘  │
│               │                    │                     ││               │
│               │                    ▼                     ││               │
│               │   ┌────────────────────────────────────┐ ││               │
│               │   │         Operation Log              │◀┘│               │
│               │   │        (Append-only)               │  │               │
│               │   │                                    │  │               │
│               │   │ [op1, op2, op3, op4, ...]          │  │               │
│               │   └────────────────┬───────────────────┘  │               │
│               │                    │                      │               │
│               │                    ▼                      │               │
│               │   ┌────────────────────────────────────┐  │               │
│               └──▶│      Document Snapshots            │  │               │
│                   │      (Periodic checkpoints)        │◀─┘               │
│                   │                                    │                  │
│                   │  Spanner: doc_id → snapshot + ops  │                  │
│                   └────────────────────────────────────┘                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Operational Transformation Example

```
┌─────────────────────────────────────────────────────────────────┐
│                  OT Example: Concurrent Inserts                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Initial document: "HELLO"                                      │
│                                                                  │
│   User A: Insert 'X' at position 2    → "HEXLLO"                │
│   User B: Insert 'Y' at position 4    → "HELLY"  (concurrent)   │
│                                                                  │
│   Without OT (naive merge):                                      │
│   Apply A's op: "HEXLLO"                                         │
│   Apply B's op at pos 4: "HEXLYLO" ❌ Wrong!                    │
│                                                                  │
│   With OT:                                                       │
│   Transform B's op against A's:                                  │
│   - A inserted at position 2                                     │
│   - B's position 4 needs to shift to 5                          │
│   - B's transformed op: Insert 'Y' at position 5                │
│                                                                  │
│   Apply A's op: "HEXLLO"                                         │
│   Apply transformed B's op: "HEXLLY" ✓ Correct!                 │
│                                                                  │
│   Code:                                                          │
│   def transform(op_a, op_b):                                     │
│       if op_a.type == INSERT and op_b.type == INSERT:           │
│           if op_a.position <= op_b.position:                     │
│               return Insert(op_b.char, op_b.position + 1)       │
│           else:                                                  │
│               return op_b  # No change needed                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interviewer Follow-up: Offline Support

> "You mentioned offline support. How would that work?"

```
Candidate Response:

"Offline editing adds significant complexity. Here's my approach:

1. **Local-first architecture**:
   - Full document stored locally (IndexedDB in browser)
   - All edits applied locally first
   - Operations queued for sync

2. **Operation buffering**:
   ┌────────────────────────────────────────┐
   │  Local Operation Buffer               │
   │                                        │
   │  Online: [op1] → send → ack → remove  │
   │                                        │
   │  Offline:                              │
   │    [op1, op2, op3, ...] → persist     │
   │                                        │
   │  Reconnect:                            │
   │    1. Fetch server state              │
   │    2. Transform local ops against     │
   │       server ops we missed            │
   │    3. Send transformed local ops      │
   │    4. Apply server ops locally        │
   └────────────────────────────────────────┘

3. **Conflict resolution**:
   - CRDTs are better here than OT
   - Each character has unique ID
   - Ordering is deterministic regardless of sync order

4. **Sync protocol**:
   - Client sends: 'last seen server version' + local ops
   - Server sends: ops since that version
   - Both transform and merge

The trade-off is complexity vs offline capability. Google Docs
uses a simplified model where offline edits are limited and
conflicts are resolved on reconnect with user intervention
for complex cases."
```

---

## Scenario 3: Design a Rate Limiter (L4 Level)

### Opening (Interviewer)

> "Design a rate limiting system for an API gateway."

### Quick Requirements Gathering

```
"Key questions:

1. What's the rate limit scope?"
   → "Per API key, 1000 requests per minute"

2. "What precision do we need?"
   → "Reasonable precision, doesn't need to be exact"

3. "Distributed system?"
   → "Yes, multiple API gateway instances"

4. "What happens when limit exceeded?"
   → "Return 429 Too Many Requests"
```

### Algorithm Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                 Rate Limiting Algorithms                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. FIXED WINDOW                                                 │
│     ┌─────────────────┬─────────────────┐                       │
│     │  Minute 1       │  Minute 2       │                       │
│     │  [1000 requests]│  [1000 requests]│                       │
│     └─────────────────┴─────────────────┘                       │
│     ⚠️ Problem: 2000 requests possible at boundary              │
│                                                                  │
│  2. SLIDING WINDOW LOG                                           │
│     Store timestamp of each request                              │
│     Count requests in last 60 seconds                            │
│     ✓ Precise                                                    │
│     ❌ High memory usage                                         │
│                                                                  │
│  3. SLIDING WINDOW COUNTER (Recommended)                        │
│     Weighted average of current and previous window             │
│     requests = prev_count * overlap% + curr_count               │
│     ✓ Low memory                                                 │
│     ✓ Smooth rate limiting                                       │
│                                                                  │
│  4. TOKEN BUCKET                                                 │
│     Bucket fills at fixed rate                                   │
│     Each request consumes a token                                │
│     ✓ Allows bursts up to bucket size                           │
│     ✓ Simple to implement                                        │
│                                                                  │
│  5. LEAKY BUCKET                                                 │
│     Requests queue, process at fixed rate                        │
│     ✓ Smooths traffic                                            │
│     ❌ May add latency                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation (Token Bucket)

```python
class DistributedRateLimiter:
    """
    Redis-based token bucket rate limiter
    """
    def __init__(self, redis_client, rate: int, capacity: int):
        self.redis = redis_client
        self.rate = rate           # Tokens per second
        self.capacity = capacity   # Max tokens

    def allow_request(self, key: str) -> bool:
        """
        Returns True if request is allowed, False otherwise.
        Uses Redis Lua script for atomicity.
        """
        lua_script = """
        local key = KEYS[1]
        local rate = tonumber(ARGV[1])
        local capacity = tonumber(ARGV[2])
        local now = tonumber(ARGV[3])
        local requested = tonumber(ARGV[4])

        -- Get current bucket state
        local bucket = redis.call('HMGET', key, 'tokens', 'last_update')
        local tokens = tonumber(bucket[1]) or capacity
        local last_update = tonumber(bucket[2]) or now

        -- Calculate tokens to add based on time passed
        local elapsed = now - last_update
        local new_tokens = math.min(capacity, tokens + (elapsed * rate))

        -- Check if request can be allowed
        local allowed = new_tokens >= requested

        if allowed then
            new_tokens = new_tokens - requested
        end

        -- Update bucket state
        redis.call('HMSET', key, 'tokens', new_tokens, 'last_update', now)
        redis.call('EXPIRE', key, 60)

        return allowed and 1 or 0
        """

        result = self.redis.eval(
            lua_script,
            1,
            key,
            self.rate,
            self.capacity,
            time.time(),
            1  # Request 1 token
        )

        return result == 1
```

### Distributed Considerations

```
"For distributed rate limiting across multiple API gateway instances:

┌─────────────────────────────────────────────────────────────────┐
│                  Distributed Rate Limiting                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Option 1: Centralized (Redis)                                  │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│   │Gateway 1│  │Gateway 2│  │Gateway 3│                         │
│   └────┬────┘  └────┬────┘  └────┬────┘                         │
│        │           │           │                                 │
│        └───────────┼───────────┘                                 │
│                    ▼                                             │
│             ┌─────────────┐                                      │
│             │   Redis     │                                      │
│             │  Cluster    │                                      │
│             └─────────────┘                                      │
│                                                                  │
│   ✓ Accurate global count                                        │
│   ❌ Redis becomes SPOF                                          │
│   ❌ Adds latency (network round-trip)                          │
│                                                                  │
│   Option 2: Local + Sync                                         │
│   Each gateway maintains local counter                           │
│   Periodically sync with central store                           │
│   ✓ Fast (local check)                                          │
│   ⚠️ Less accurate (eventual consistency)                       │
│                                                                  │
│   Option 3: Sticky Sessions                                      │
│   Route same API key to same gateway                             │
│   ✓ Simple local rate limiting                                   │
│   ❌ Uneven load distribution                                    │
│                                                                  │
│   Recommendation: Option 1 with circuit breaker                  │
│   - Use Redis for accurate counting                              │
│   - Fall back to local limiting if Redis unavailable            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘"
```

---

## Interview Tips Summary

### Do's

```
✓ Ask clarifying questions before designing
✓ State your assumptions explicitly
✓ Start with high-level, then zoom in
✓ Discuss trade-offs for each decision
✓ Draw diagrams to visualize architecture
✓ Think out loud - share your reasoning
✓ Quantify: users, requests/sec, storage
✓ Consider failure modes
```

### Don'ts

```
✗ Jump straight to implementation details
✗ Design in silence
✗ Ignore hints from interviewer
✗ Over-engineer from the start
✗ Forget about operational concerns
✗ Give up when stuck - reason through it
✗ Dismiss a design approach without explaining why
```

### Time Management

```
┌─────────────────────────────────────────────────────────────────┐
│                   45-Minute Interview Timeline                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  0:00 - 0:05  │  Problem clarification, requirements            │
│  0:05 - 0:10  │  Scale estimation, non-functional requirements  │
│  0:10 - 0:25  │  High-level design, component discussion        │
│  0:25 - 0:40  │  Deep dive into 1-2 key components              │
│  0:40 - 0:45  │  Wrap-up, questions for interviewer             │
│                                                                  │
│  Key: Don't spend more than 5 min on any single topic unless   │
│       the interviewer is driving the deep dive.                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Related Topics

- [[00_google_overview|Google Prep Overview]]
- [[01_interview_format|Interview Format]]
- [[03_google_case_studies|Case Studies]]

---

**Tags**: #google #mock-interview #practice #notifications #collaborative-editing #rate-limiter
