# Google System Design Interview Prep

> Specialized preparation for Google's system design interviews, covering L4 through L6+ levels.

## Why Google is Different

Google's system design interviews have unique characteristics:

1. **Scale expectations**: Google operates at massive scale - billions of users, petabytes of data
2. **Infrastructure knowledge**: Familiarity with Google's public papers is valuable
3. **Ambiguity tolerance**: Questions are intentionally open-ended
4. **Trade-off discussions**: Strong emphasis on justifying decisions
5. **Follow-up depth**: Interviewers dive deep into specific areas

---

## Interview Levels

| Level | Title | Design Scope | Expectations |
|-------|-------|--------------|--------------|
| **L4** | Software Engineer | Component design | Solid fundamentals, guided discussion |
| **L5** | Senior SWE | System design | Lead the discussion, identify trade-offs |
| **L6** | Staff SWE | Large-scale systems | Strategic thinking, cross-team impact |
| **L7+** | Principal+ | Organization-wide | Industry-level impact, novel solutions |

---

## Section Contents

### [1. Interview Format](01_interview_format.md)
- 45-minute structure
- What interviewers look for
- Level-specific expectations
- Evaluation rubric

### [2. Google-Specific Topics](02_google_topics.md)
- Spanner & global databases
- Bigtable & wide-column stores
- MapReduce & Dataflow
- Borg & Kubernetes origins
- Pub/Sub messaging

### [3. Google Case Studies](03_google_case_studies.md)
- Design Google Search
- Design Google Maps
- Design Google Drive
- Design Gmail
- Design YouTube
- Design Google Photos

### [4. Infrastructure Deep Dives](04_infrastructure_deep_dives.md)
- Colossus (GFS successor)
- Chubby (distributed locking)
- Zanzibar (authorization)
- Spanner internals

### [5. Mock Interview Scenarios](05_mock_scenarios.md)
- Full interview walkthroughs
- Common follow-ups
- Handling ambiguity

---

## Google's Published Papers (Essential Reading)

| Paper | Year | Key Concepts |
|-------|------|--------------|
| **GFS** | 2003 | Distributed file system, chunk servers |
| **MapReduce** | 2004 | Distributed processing paradigm |
| **Bigtable** | 2006 | Wide-column store, tablet servers |
| **Chubby** | 2006 | Distributed lock service, Paxos |
| **Spanner** | 2012 | Globally distributed database, TrueTime |
| **F1** | 2013 | Distributed SQL on Spanner |
| **Borg** | 2015 | Container orchestration (Kubernetes origin) |
| **Zanzibar** | 2019 | Global authorization system |

---

## Key Differences from Other FAANG

| Aspect | Google | Others |
|--------|--------|--------|
| **Scale** | Global-first thinking | Often regional focus |
| **Infrastructure** | Build from scratch | Use existing solutions |
| **Consistency** | Strong consistency preference | Often eventual consistency |
| **Data** | Structured data, SQL at scale | NoSQL more common |
| **Follow-ups** | Deep technical dives | Broader coverage |

---

## Quick Prep Checklist

### Must Know
- [ ] CAP theorem and Google's approach (Spanner)
- [ ] Consistent hashing and data distribution
- [ ] Pub/Sub patterns at scale
- [ ] Global load balancing
- [ ] Storage hierarchy (RAM → SSD → HDD → Tape)

### Should Know
- [ ] How Bigtable differs from traditional DBs
- [ ] MapReduce vs. streaming (Dataflow)
- [ ] Borg's influence on Kubernetes
- [ ] TrueTime and GPS/atomic clocks

### Nice to Know
- [ ] Colossus architecture
- [ ] Zanzibar's relationship model
- [ ] Dremel and BigQuery internals
- [ ] Monarch (monitoring system)

---

## Study Timeline

### 1 Week Intensive
- Day 1-2: Interview format + estimation practice
- Day 3-4: Google-specific topics overview
- Day 5-6: 3 key case studies (Search, Drive, YouTube)
- Day 7: Mock interviews + review

### 2 Week Comprehensive
- Week 1: All case studies + infrastructure deep dives
- Week 2: Mock interviews + paper reading + weak area focus

---

## Related Topics

- [[../HLD/00_hld|High Level Design]]
- [[../Interview_Prep/00_interview_prep|General Interview Prep]]
- [[../Fundamentals/00_fundamentals|System Design Fundamentals]]

---

**Tags**: #google #system-design #interview #faang #l5 #l6
