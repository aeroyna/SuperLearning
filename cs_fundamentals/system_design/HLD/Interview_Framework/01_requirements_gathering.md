# Requirements Gathering

The first 5 minutes of a system design interview are crucial. Good requirement gathering shows maturity and sets you up for success.

---

## Why Requirements Matter

```
"Design a chat application"

Without clarification:
- You assume text-only, interviewer expects video
- You design for 1000 users, they want 1 billion
- You focus on 1-on-1, they want group chat

With clarification:
- Clear scope
- Appropriate scale
- Aligned expectations
```

---

## Functional Requirements

### What the system should do

**Template Questions:**
```
1. Who are the users?
   - Consumers? Businesses? Internal?
   - Mobile? Web? Both?

2. What are the core features?
   - "What are the must-have features for MVP?"
   - "What can we deprioritize for this discussion?"

3. How do users interact?
   - "Walk me through a typical user flow"

4. What data do we need to handle?
   - Text? Images? Videos?
   - User-generated? System-generated?
```

### Example: Chat Application

```
Functional Requirements:

Must Have:
✓ Send/receive text messages
✓ 1-on-1 messaging
✓ Group messaging (up to 100 members)
✓ Message delivery status (sent, delivered, read)
✓ Online/offline status

Nice to Have (if time):
○ Message reactions
○ File/image sharing
○ Voice/video calls

Out of Scope:
✗ Message encryption (E2E)
✗ Message search
✗ Bot integrations
```

---

## Non-Functional Requirements (NFRs)

### Scale
```
- How many users? (DAU, MAU)
- How much data? (messages/day, storage growth)
- Traffic patterns? (steady vs bursty)

Example:
"Let's assume 100 million DAU, each user sends 50 messages/day"
```

### Performance
```
- Latency requirements?
  - "Message delivery should be < 100ms"
  - "Feed should load in < 500ms"

- Throughput requirements?
  - "System should handle 10K messages/second"
```

### Availability
```
- How many 9s?
  - 99.9% = 8.76 hours downtime/year
  - 99.99% = 52.6 minutes downtime/year

- "Is this a critical system? Should we prioritize availability?"
```

### Consistency
```
- Strong consistency needed?
  - Banking: Yes
  - Social feed: No (eventual OK)

- "Is it OK if messages appear slightly out of order briefly?"
```

### Other NFRs
```
- Security (authentication, authorization)
- Compliance (GDPR, data residency)
- Cost constraints
- Existing infrastructure
```

---

## Constraint Questions

```
Technical Constraints:
- "Are we building from scratch or integrating with existing systems?"
- "Any specific technology requirements?"
- "Cloud provider preference?"

Business Constraints:
- "What's the timeline?"
- "Team size for this project?"
- "Budget considerations?"
```

---

## Clarification Template

```markdown
"Before I start designing, let me clarify some requirements.

**Functional:**
1. [Ask about core feature 1]
2. [Ask about core feature 2]
3. [Confirm scope boundaries]

**Scale:**
- "How many users are we targeting?"
- "What's the expected data volume?"

**Performance:**
- "What latency is acceptable for [key operation]?"

**Priorities:**
- "Should I prioritize [availability/consistency/partition tolerance]?"

**Scope:**
- "Should I focus on [specific area] or cover the full system?"

Let me summarize what I heard: [Restate requirements]
Does this sound right?"
```

---

## Red Flags to Clarify

| Statement | Clarify |
|-----------|---------|
| "Design X" | What aspects of X? |
| "Handle lots of users" | How many exactly? |
| "Fast response" | What latency is acceptable? |
| "Store data" | How much? How long? |
| "Real-time" | What does real-time mean here? |
| "Scalable" | Scale to what level? |

---

## Example Dialogue

```
Interviewer: "Design a notification system"

You: "Great! Let me clarify some requirements.

First, what types of notifications?
- Push notifications to mobile?
- Email?
- In-app notifications?
- SMS?"

Interviewer: "All of them."

You: "Got it. What's the scale we're targeting?
- How many users?
- Notifications per user per day?"

Interviewer: "100 million users, average 10 notifications per day."

You: "That's 1 billion notifications per day. Are there peak times?
Like sales events where volume might spike 10x?"

Interviewer: "Yes, we have flash sales occasionally."

You: "Understood. For latency, some notifications need to be
real-time (order updates) vs batch (weekly digest). Should I
consider both?"

Interviewer: "Yes, prioritize real-time but mention batch."

You: "Perfect. Let me summarize:
- Multi-channel: push, email, in-app, SMS
- 100M users, 1B notifications/day, 10x peaks
- Mix of real-time and batch
- I'll focus on the real-time path

Does this capture the scope correctly?"
```

---

## Common Gotchas

1. **Assuming too much**: State assumptions explicitly
2. **Forgetting mobile**: Many systems need mobile support
3. **Ignoring edge cases**: What about bad input? Duplicate requests?
4. **Missing geographic distribution**: Users in different regions?
5. **Not prioritizing**: You can't cover everything in 45 minutes
