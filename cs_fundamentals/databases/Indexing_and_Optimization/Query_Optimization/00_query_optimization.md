# Query Optimization

## Introduction

Query optimization is the process of selecting the most efficient execution plan for a SQL query. The query optimizer is one of the most complex components of a database system, responsible for transforming declarative SQL into efficient procedural execution plans.

## Query Processing Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                  Query Processing Pipeline                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  SQL Query: SELECT * FROM orders o                          │
│             JOIN customers c ON o.customer_id = c.id        │
│             WHERE c.country = 'USA' AND o.amount > 100      │
│                              │                               │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 1. PARSER                                              │  │
│  │    Lexical analysis, syntax checking                   │  │
│  │    Output: Parse Tree (AST)                            │  │
│  └───────────────────────────────────────────────────────┘  │
│                              │                               │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 2. SEMANTIC ANALYZER                                   │  │
│  │    Validate tables, columns, types                     │  │
│  │    Resolve aliases, check permissions                  │  │
│  │    Output: Validated Query Tree                        │  │
│  └───────────────────────────────────────────────────────┘  │
│                              │                               │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 3. QUERY REWRITER                                      │  │
│  │    View expansion, subquery flattening                 │  │
│  │    Predicate pushdown, common subexpression elim       │  │
│  │    Output: Rewritten Query Tree                        │  │
│  └───────────────────────────────────────────────────────┘  │
│                              │                               │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 4. QUERY OPTIMIZER                                     │  │
│  │    Generate possible plans                             │  │
│  │    Estimate costs using statistics                     │  │
│  │    Select optimal plan                                 │  │
│  │    Output: Optimal Execution Plan                      │  │
│  └───────────────────────────────────────────────────────┘  │
│                              │                               │
│                              ▼                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │ 5. QUERY EXECUTOR                                      │  │
│  │    Execute plan operators                              │  │
│  │    Manage memory, disk I/O                             │  │
│  │    Output: Query Results                               │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Optimization Approaches

### Rule-Based vs Cost-Based

```
┌─────────────────────────────────────────────────────────────┐
│           Rule-Based vs Cost-Based Optimization              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  RULE-BASED (Heuristic):                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Apply fixed transformation rules                   │    │
│  │ • Always use index if available                      │    │
│  │ • Push selections down                               │    │
│  │ • Simple but inflexible                              │    │
│  │ • Deprecated in modern databases                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COST-BASED:                                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Estimate cost of each plan alternative             │    │
│  │ • Use statistics about data                          │    │
│  │ • Choose lowest cost plan                            │    │
│  │ • More accurate but complex                          │    │
│  │ • Used by PostgreSQL, MySQL, Oracle, SQL Server      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ADAPTIVE:                                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Start with cost-based plan                         │    │
│  │ • Collect runtime statistics                         │    │
│  │ • Adjust plan mid-execution if estimates wrong       │    │
│  │ • Oracle 12c+, SQL Server Adaptive joins             │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Query Plan Representation

### Operator Tree

```
Query: SELECT c.name, SUM(o.amount)
       FROM customers c JOIN orders o ON c.id = o.customer_id
       WHERE c.country = 'USA'
       GROUP BY c.name
       HAVING SUM(o.amount) > 1000

┌─────────────────────────────────────────────────────────────┐
│                     Logical Plan Tree                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                    ┌─────────────┐                          │
│                    │   Project   │                          │
│                    │ (name, sum) │                          │
│                    └──────┬──────┘                          │
│                           │                                  │
│                    ┌──────┴──────┐                          │
│                    │   Filter    │                          │
│                    │ sum > 1000  │                          │
│                    └──────┬──────┘                          │
│                           │                                  │
│                    ┌──────┴──────┐                          │
│                    │  GroupBy    │                          │
│                    │  (name)     │                          │
│                    │  SUM(amount)│                          │
│                    └──────┬──────┘                          │
│                           │                                  │
│                    ┌──────┴──────┐                          │
│                    │    Join     │                          │
│                    │ c.id=o.cust │                          │
│                    └──────┬──────┘                          │
│                    ┌──────┴──────┐                          │
│              ┌─────┴─────┐ ┌─────┴─────┐                    │
│              │  Filter   │ │   Scan    │                    │
│              │ country=  │ │  orders   │                    │
│              │  'USA'    │ └───────────┘                    │
│              └─────┬─────┘                                  │
│              ┌─────┴─────┐                                  │
│              │   Scan    │                                  │
│              │ customers │                                  │
│              └───────────┘                                  │
└─────────────────────────────────────────────────────────────┘
```

## Topics Covered

1. **[Cost-Based Optimization](01_cost_based_optimization.md)** - Cost models and estimation
2. **[Query Execution Plans](02_query_execution_plans.md)** - Reading and understanding plans
3. **[Join Algorithms](03_join_algorithms.md)** - Nested loop, hash, merge joins
4. **[Statistics and Cardinality](04_statistics_and_cardinality.md)** - Histograms and estimation

## Common Optimization Techniques

### Predicate Pushdown

```
Before:
┌─────────────────────────────────────────────────────────────┐
│                    ┌─────────────┐                          │
│                    │   Filter    │                          │
│                    │ country='US'│                          │
│                    └──────┬──────┘                          │
│                           │                                  │
│                    ┌──────┴──────┐                          │
│                    │    Join     │  ← Process all rows     │
│                    │             │    then filter           │
│                    └──────┬──────┘                          │
│              ┌────────────┴────────────┐                    │
│              ▼                         ▼                    │
│         [customers]               [orders]                  │
│         1M rows                   10M rows                  │
└─────────────────────────────────────────────────────────────┘

After Pushdown:
┌─────────────────────────────────────────────────────────────┐
│                    ┌─────────────┐                          │
│                    │    Join     │  ← Only 50K × 500K      │
│                    │             │    = 50K result rows     │
│                    └──────┬──────┘                          │
│              ┌────────────┴────────────┐                    │
│              ▼                         ▼                    │
│         ┌─────────┐              ┌─────────┐               │
│         │ Filter  │              │  Scan   │               │
│         │country= │              │ orders  │               │
│         │ 'USA'   │              │(indexed)│               │
│         └────┬────┘              └─────────┘               │
│         ┌────┴────┐                                        │
│         │customers│  ← 50K US customers                    │
│         └─────────┘                                        │
└─────────────────────────────────────────────────────────────┘
```

### Join Reordering

```
Query: SELECT * FROM A JOIN B JOIN C JOIN D
       WHERE A.x = B.y AND B.z = C.w AND C.q = D.r

Possible Join Orders (for 4 tables):
┌─────────────────────────────────────────────────────────────┐
│  Number of orderings: (2n-2)! / (n-1)! = 4! = 24           │
│                                                              │
│  But with only connected joins considered:                   │
│                                                              │
│  Order 1: ((A ⋈ B) ⋈ C) ⋈ D                                │
│  Order 2: (A ⋈ (B ⋈ C)) ⋈ D                                │
│  Order 3: ((A ⋈ B) ⋈ D) ⋈ C  ← Invalid (C not connected)  │
│  ...                                                         │
│                                                              │
│  Optimizer uses dynamic programming to find best order:     │
│                                                              │
│  Cost analysis:                                              │
│  • |A| = 1000, |B| = 10000, |C| = 100, |D| = 500           │
│  • Selectivity(A⋈B) = 0.1                                   │
│  • Selectivity(B⋈C) = 0.01                                  │
│                                                              │
│  Best order might be: (C ⋈ B) ⋈ A ⋈ D                      │
│  Start with smallest result, grow incrementally             │
└─────────────────────────────────────────────────────────────┘
```

### Subquery Decorrelation

```sql
-- Correlated subquery (slow):
SELECT *
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id  -- Correlation
);

-- Decorrelated (optimized):
SELECT e.*
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) as avg_sal
    FROM employees
    GROUP BY department_id
) dept_avg ON e.department_id = dept_avg.department_id
WHERE e.salary > dept_avg.avg_sal;
```

## Plan Caching

```
┌─────────────────────────────────────────────────────────────┐
│                     Plan Cache Flow                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Query: SELECT * FROM users WHERE id = ?                    │
│                                                              │
│  ┌─────────────────┐                                        │
│  │ Compute Query   │                                        │
│  │ Hash/Signature  │                                        │
│  └────────┬────────┘                                        │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────┐     ┌─────────────────┐               │
│  │ Cache Lookup    │────→│ Cache Hit?      │               │
│  └─────────────────┘     └────────┬────────┘               │
│                                   │                          │
│              ┌────────────────────┴────────────────────┐    │
│              │ YES                                 NO  │    │
│              ▼                                         ▼    │
│  ┌───────────────────┐                    ┌──────────────┐ │
│  │ Reuse Cached Plan │                    │ Optimize &   │ │
│  │ (skip optimization)│                    │ Cache Plan   │ │
│  └───────────────────┘                    └──────────────┘ │
│                                                              │
│  Prepared Statements (Parameterized):                       │
│  • Parse once, execute many times                           │
│  • Plan cached with parameter placeholders                  │
│  • May use generic plan for all values                      │
│                                                              │
│  PostgreSQL: plan_cache_mode = auto|force_generic|force_custom
│  MySQL: query_cache (deprecated), prepared statement cache  │
└─────────────────────────────────────────────────────────────┘
```

## Optimizer Hints

### MySQL Hints

```sql
-- Force index usage
SELECT * FROM orders USE INDEX (idx_customer_id)
WHERE customer_id = 100;

-- Force join order
SELECT /*+ JOIN_ORDER(c, o, p) */
    c.name, o.order_date, p.name
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN products p ON o.product_id = p.id;

-- Force join algorithm
SELECT /*+ HASH_JOIN(o, c) */
    * FROM orders o JOIN customers c ON o.customer_id = c.id;

-- Optimizer switch
SET optimizer_switch='index_merge=on,index_merge_union=on';
```

### PostgreSQL Hints (pg_hint_plan)

```sql
-- Install extension
CREATE EXTENSION pg_hint_plan;

-- Force sequential scan
/*+ SeqScan(users) */
SELECT * FROM users WHERE status = 'active';

-- Force index scan
/*+ IndexScan(orders idx_orders_date) */
SELECT * FROM orders WHERE order_date > '2024-01-01';

-- Force join method
/*+ HashJoin(orders customers) */
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id;

-- Combine hints
/*+
  HashJoin(o c)
  IndexScan(o idx_orders_date)
  SeqScan(c)
  Rows(o c #1000)
*/
SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id
WHERE o.order_date > '2024-01-01';
```

## Monitoring and Troubleshooting

### Identifying Slow Queries

```sql
-- PostgreSQL: Find slow queries
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- MySQL: Slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- seconds
-- Check: /var/log/mysql/slow.log

-- MySQL: Performance Schema
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```

### Common Performance Issues

```
┌─────────────────────────────────────────────────────────────┐
│              Common Query Performance Issues                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Missing Indexes                                          │
│     Symptom: Seq Scan on large table                        │
│     Solution: Create appropriate index                       │
│                                                              │
│  2. Poor Statistics                                          │
│     Symptom: Wrong join order, bad row estimates            │
│     Solution: ANALYZE table, increase statistics target     │
│                                                              │
│  3. N+1 Query Pattern                                       │
│     Symptom: Many small queries instead of one JOIN         │
│     Solution: Use JOIN or batch queries                      │
│                                                              │
│  4. Implicit Type Conversion                                │
│     Symptom: Index not used due to type mismatch            │
│     Solution: Match column and parameter types               │
│                                                              │
│  5. Functions on Indexed Columns                            │
│     Symptom: WHERE YEAR(date) = 2024 → no index            │
│     Solution: date >= '2024-01-01' AND date < '2025-01-01' │
│                                                              │
│  6. OR Conditions                                            │
│     Symptom: No index usage with OR                         │
│     Solution: UNION of separate queries, or IN clause       │
│                                                              │
│  7. Correlated Subqueries                                   │
│     Symptom: Subquery runs once per outer row               │
│     Solution: Rewrite as JOIN                                │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

- Query optimization transforms declarative SQL into efficient execution
- Cost-based optimization uses statistics to estimate plan costs
- Understanding execution plans is crucial for performance tuning
- Predicate pushdown and join ordering are key optimizations
- Plan caching avoids repeated optimization overhead
- Hints should be used sparingly when optimizer makes wrong choices
