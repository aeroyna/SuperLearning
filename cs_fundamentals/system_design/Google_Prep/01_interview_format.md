# Google Interview Format

> Understanding Google's unique system design interview structure and expectations.

## Interview Structure

### Standard 45-Minute Format

```
┌─────────────────────────────────────────────────────────────────┐
│  0-5 min   │ Problem Introduction & Clarification              │
├─────────────────────────────────────────────────────────────────┤
│  5-10 min  │ Requirements Gathering & Scope Definition         │
├─────────────────────────────────────────────────────────────────┤
│  10-25 min │ High-Level Design & Component Discussion          │
├─────────────────────────────────────────────────────────────────┤
│  25-40 min │ Deep Dive into 1-2 Components                     │
├─────────────────────────────────────────────────────────────────┤
│  40-45 min │ Wrap-up & Candidate Questions                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## What Interviewers Look For

### 1. Problem Decomposition
```
✓ Breaking complex problems into manageable pieces
✓ Identifying the core challenges
✓ Prioritizing what to solve first
```

### 2. Trade-off Analysis
```
✓ Articulating pros and cons of different approaches
✓ Justifying decisions with reasoning
✓ Understanding when to use what
```

### 3. Scale Awareness
```
✓ Thinking in terms of billions of users
✓ Understanding data growth implications
✓ Planning for 10x scale from day one
```

### 4. Technical Depth
```
✓ Deep knowledge in at least one area
✓ Understanding implementation details
✓ Knowing failure modes and recovery
```

---

## Level-Specific Expectations

### L4 (Software Engineer)

**What's Expected:**
- Solid fundamentals of distributed systems
- Can design a single service end-to-end
- Needs some guidance on scope
- Understands basic trade-offs

**Sample Question Complexity:**
- Design a URL shortener
- Design a rate limiter
- Design a cache system

**Key Signals:**
- Asks clarifying questions
- Produces reasonable design
- Identifies obvious bottlenecks

### L5 (Senior Software Engineer)

**What's Expected:**
- Leads the discussion independently
- Designs multi-service systems
- Proactively identifies edge cases
- Deep dive into at least one area

**Sample Question Complexity:**
- Design Google Drive
- Design a notification system
- Design a feed ranking system

**Key Signals:**
- Drives the conversation
- Makes and justifies trade-offs
- Anticipates scaling challenges
- Considers operational aspects

### L6 (Staff Software Engineer)

**What's Expected:**
- Designs organization-scale systems
- Considers cross-team implications
- Thinks about long-term evolution
- Identifies novel solutions

**Sample Question Complexity:**
- Design Google Search
- Design a global payment system
- Design a real-time collaboration system

**Key Signals:**
- Strategic thinking
- Considers organizational impact
- Innovative approaches
- Anticipates future requirements

### L7+ (Principal/Distinguished)

**What's Expected:**
- Industry-level impact thinking
- Novel architectural patterns
- Multi-year roadmap considerations
- Mentoring/teaching during interview

**Key Signals:**
- Teaches the interviewer something
- Proposes innovative solutions
- Considers industry-wide implications

---

## Google's Evaluation Criteria

### Technical Skills (Primary)

| Dimension | What They Assess |
|-----------|------------------|
| **Coding** | Not in system design, but influences thinking |
| **System Design** | Architecture, scalability, trade-offs |
| **Problem Solving** | Analytical thinking, creativity |

### Behavioral (Secondary in SD)

| Dimension | What They Assess |
|-----------|------------------|
| **Googleyness** | Collaboration, comfort with ambiguity |
| **Leadership** | Driving discussions, mentoring signals |

---

## Common Interview Patterns

### Pattern 1: The Open-Ended Start
```
Interviewer: "Design a system like Google Photos"

What they want:
- You to ask clarifying questions
- You to define scope
- You to drive the discussion
```

### Pattern 2: The Constraint Addition
```
Mid-interview: "Now assume you need to support
offline access for all users globally"

What they want:
- Adaptation of existing design
- Understanding of new trade-offs
- Clear communication of changes
```

### Pattern 3: The Deep Dive
```
"Let's dive deeper into how you'd handle
the image processing pipeline"

What they want:
- Technical depth
- Implementation awareness
- Failure handling
```

### Pattern 4: The Scale Challenge
```
"How would this change if we had 10x the users?"

What they want:
- Scalability thinking
- Bottleneck identification
- Evolution strategy
```

---

## How to Structure Your Response

### Opening (0-5 min)
```markdown
"Before I start designing, I'd like to clarify a few things:
1. What's our target scale? Users, data volume?
2. What are the most critical features?
3. Any specific constraints I should know about?"
```

### Requirements (5-10 min)
```markdown
"Based on our discussion, let me summarize:

Functional Requirements:
- Users can upload/download photos
- Photos are organized by date/location
- Sharing with specific users

Non-Functional Requirements:
- 1B users, 10M DAU
- 99.9% availability
- < 200ms latency for viewing
- Durability: never lose a photo

Out of Scope (for now):
- Video processing
- Advanced ML features
```

### High-Level Design (10-25 min)
```markdown
"Let me draw the high-level architecture:

[Draw diagram]

Key components:
1. Upload Service - handles incoming photos
2. Processing Pipeline - generates thumbnails
3. Storage Layer - for raw and processed images
4. Metadata Service - for organization
5. CDN - for fast delivery

Let me explain how data flows through the system..."
```

### Deep Dive (25-40 min)
```markdown
"I'd like to dive into [component] because
it's the most critical for [reason].

Here's how I'd implement it:
- Data model: ...
- API design: ...
- Scaling strategy: ...
- Failure handling: ..."
```

---

## Red Flags to Avoid

### ❌ Don't Do This

1. **Jumping to solution immediately**
   - Without understanding requirements

2. **Being silent while thinking**
   - Always verbalize your thought process

3. **Ignoring interviewer hints**
   - They're trying to help you

4. **Over-engineering from the start**
   - Start simple, then scale

5. **Not asking about scale**
   - Scale changes everything at Google

6. **Saying "I don't know" and stopping**
   - Instead: "I'm not certain, but I'd approach it by..."

### ✓ Do This Instead

1. **Ask clarifying questions first**
2. **Think out loud**
3. **Pick up on interviewer cues**
4. **Start with MVP, then iterate**
5. **Always quantify scale**
6. **Make educated guesses with reasoning**

---

## Sample Questions by Level

### L4 Questions
- Design a key-value store
- Design a rate limiter
- Design a URL shortener
- Design a simple search autocomplete

### L5 Questions
- Design Google Drive
- Design a notification system
- Design a web crawler
- Design a ticket booking system

### L6 Questions
- Design Google Search
- Design YouTube's recommendation system
- Design a global payment system
- Design a real-time collaboration system

---

## Related Topics

- [[00_google_overview|Google Prep Overview]]
- [[02_google_topics|Google-Specific Topics]]
- [[05_mock_scenarios|Mock Interview Scenarios]]

---

**Tags**: #google #interview #format #expectations #levels
