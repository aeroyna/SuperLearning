# MySQL Query Optimization

## Overview

Query optimization is essential for maintaining database performance as data grows. Understanding how MySQL executes queries and using the right tools to analyze and improve them is a critical skill for developers and DBAs.

This section covers:

1. **[EXPLAIN and Query Plans](01_explain_and_query_plans.md)** - Understanding query execution plans
2. **[Query Profiling](02_query_profiling.md)** - Measuring query performance
3. **[Slow Query Log](03_slow_query_log.md)** - Identifying problematic queries
4. **[Optimization Techniques](04_optimization_techniques.md)** - Practical optimization strategies

---

## Query Optimization Process

```
┌─────────────────────────────────────────────────────────────────────┐
│                   Query Optimization Workflow                        │
│                                                                      │
│  1. IDENTIFY                                                         │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ • Slow query log                                              │   │
│  │ • Performance Schema                                          │   │
│  │ • Application monitoring                                      │   │
│  │ • User complaints                                             │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              ↓                                       │
│  2. ANALYZE                                                          │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ • EXPLAIN / EXPLAIN ANALYZE                                   │   │
│  │ • Query profiling                                             │   │
│  │ • Index analysis                                              │   │
│  │ • Table statistics                                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              ↓                                       │
│  3. OPTIMIZE                                                         │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ • Rewrite query                                               │   │
│  │ • Add/modify indexes                                          │   │
│  │ • Restructure data                                            │   │
│  │ • Configure MySQL settings                                    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                              ↓                                       │
│  4. VERIFY                                                           │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ • Re-run EXPLAIN                                              │   │
│  │ • Measure execution time                                      │   │
│  │ • Monitor in production                                       │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Metrics to Monitor

| Metric | Good | Concerning | Critical |
|--------|------|------------|----------|
| Query time | < 100ms | 100ms - 1s | > 1s |
| Rows examined | ≈ rows returned | 10x rows returned | 100x+ rows |
| Index usage | Yes | Partial | Full table scan |
| Temp tables | None | On disk rarely | Frequent disk temps |

---

## Quick Reference

```sql
-- Analyze query execution
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Check table indexes
SHOW INDEX FROM users;

-- View query statistics
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;

-- Enable slow query log
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 1;
```

---

## Learning Path

1. Master EXPLAIN output interpretation
2. Learn profiling tools and techniques
3. Set up and analyze slow query logs
4. Apply optimization techniques systematically
