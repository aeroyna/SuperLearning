# Statistics and Cardinality Estimation

## Introduction

Database statistics are metadata about table contents that the query optimizer uses to estimate costs and choose execution plans. Accurate statistics are essential for good query plans. Cardinality estimation—predicting how many rows each operation will produce—is the foundation of cost-based optimization.

## Types of Statistics

```
┌─────────────────────────────────────────────────────────────┐
│                    Statistics Overview                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TABLE-LEVEL STATISTICS:                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Row count (n_live_tup)                            │    │
│  │ • Page count (relpages)                             │    │
│  │ • Average row width                                  │    │
│  │ • Dead tuple count                                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COLUMN-LEVEL STATISTICS:                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Null fraction                                      │    │
│  │ • Number of distinct values (n_distinct)            │    │
│  │ • Most common values (MCV) and frequencies          │    │
│  │ • Histogram bounds (for range queries)              │    │
│  │ • Correlation (physical vs logical ordering)        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  INDEX STATISTICS:                                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Index size (pages)                                 │    │
│  │ • Number of leaf pages                               │    │
│  │ • Index height/depth                                 │    │
│  │ • Clustering factor (how ordered)                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Histograms

### Equi-Width Histogram

```
┌─────────────────────────────────────────────────────────────┐
│                   Equi-Width Histogram                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Column: age (range 0-100)                                   │
│  Buckets: 10 (width = 10 each)                              │
│                                                              │
│       Count                                                  │
│       │                                                      │
│  1200 ┼      ╔═══╗                                          │
│  1000 ┼   ╔══╣   ╠══╗                                       │
│   800 ┼╔══╣  ║   ║  ╠══╗                                    │
│   600 ┼║  ║  ║   ║  ║  ╠══╗                                 │
│   400 ┼║  ║  ║   ║  ║  ║  ╠══╗  ╔══╗                        │
│   200 ┼║  ║  ║   ║  ║  ║  ║  ╠══╣  ╠══╗                     │
│     0 ┼╩══╩══╩═══╩══╩══╩══╩══╩══╩══╩══╩─► Age              │
│        0  10 20  30 40 50 60 70 80 90 100                   │
│                                                              │
│  Problem: Uneven distribution within buckets                │
│  Bucket [20-30] has 1000 values but:                        │
│    - 900 might be at age 25                                 │
│    - Only 100 spread across 20-24 and 26-29                 │
│  Query "age = 25" estimates 100 (1000/10), actual = 900    │
└─────────────────────────────────────────────────────────────┘
```

### Equi-Height (Equi-Depth) Histogram

```
┌─────────────────────────────────────────────────────────────┐
│                  Equi-Height Histogram                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Each bucket contains approximately same number of rows     │
│                                                              │
│  Total rows: 10,000, Buckets: 10                            │
│  Each bucket: ~1,000 rows                                    │
│                                                              │
│  Bucket │ Range      │ Count │ Bounds                      │
│  ───────┼────────────┼───────┼──────────────────────────── │
│    1    │ 0 - 18     │ 1000  │ narrow (few distinct)      │
│    2    │ 18 - 22    │ 1000  │ narrow (high density)      │
│    3    │ 22 - 25    │ 1000  │ very narrow (cluster)      │
│    4    │ 25 - 35    │ 1000  │ wider (sparser)            │
│    5    │ 35 - 45    │ 1000  │ ...                        │
│    6    │ 45 - 55    │ 1000  │                            │
│    7    │ 55 - 65    │ 1000  │                            │
│    8    │ 65 - 75    │ 1000  │                            │
│    9    │ 75 - 90    │ 1000  │ wider (sparser)            │
│   10    │ 90 - 100   │ 1000  │                            │
│                                                              │
│  Advantage: Narrow buckets where data is dense              │
│  Range query "age BETWEEN 22 AND 25": ~1 bucket = 1000 rows │
│  More accurate than equi-width for skewed data              │
└─────────────────────────────────────────────────────────────┘
```

### Most Common Values (MCV)

```
┌─────────────────────────────────────────────────────────────┐
│                   Most Common Values                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  For columns with popular values (skewed distribution)      │
│                                                              │
│  Column: country                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Most Common Values    │ Frequency                      │ │
│  │───────────────────────┼────────────────────────────────│ │
│  │ 'USA'                 │ 0.35 (35%)                     │ │
│  │ 'China'               │ 0.20 (20%)                     │ │
│  │ 'India'               │ 0.15 (15%)                     │ │
│  │ 'UK'                  │ 0.08 (8%)                      │ │
│  │ 'Germany'             │ 0.05 (5%)                      │ │
│  │ ... (remaining 195)   │ 0.17 (17%) total              │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Query: WHERE country = 'USA'                               │
│  Cardinality estimate: 0.35 × total_rows = 0.35 × 1M = 350K │
│                                                              │
│  Query: WHERE country = 'Liechtenstein'                     │
│  Not in MCV list, estimate: (1 - sum(mcv_freq)) / n_distinct │
│  = 0.17 / 195 = 0.00087 = 870 rows                          │
│                                                              │
│  PostgreSQL: default_statistics_target = 100 (MCVs stored)  │
└─────────────────────────────────────────────────────────────┘
```

## Collecting Statistics

### PostgreSQL

```sql
-- Manual analysis (full table)
ANALYZE users;

-- Analyze specific columns
ANALYZE users (email, status);

-- Analyze entire database
ANALYZE;

-- Verbose output
ANALYZE VERBOSE users;

-- View statistics
SELECT
    attname,
    n_distinct,
    most_common_vals,
    most_common_freqs,
    histogram_bounds
FROM pg_stats
WHERE tablename = 'users';

-- Increase statistics target for important columns
ALTER TABLE users ALTER COLUMN status SET STATISTICS 1000;
ANALYZE users;

-- Check when stats were last updated
SELECT
    schemaname,
    relname,
    last_analyze,
    last_autoanalyze,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE relname = 'users';
```

### MySQL

```sql
-- Analyze table (updates index statistics)
ANALYZE TABLE users;

-- View table statistics
SELECT
    table_name,
    table_rows,
    avg_row_length,
    data_length,
    index_length
FROM information_schema.tables
WHERE table_name = 'users';

-- View index statistics
SHOW INDEX FROM users;

-- InnoDB persistent statistics
SELECT * FROM mysql.innodb_table_stats
WHERE table_name = 'users';

SELECT * FROM mysql.innodb_index_stats
WHERE table_name = 'users';

-- Configure histogram statistics (MySQL 8.0+)
ANALYZE TABLE users UPDATE HISTOGRAM ON country WITH 100 BUCKETS;

-- View histograms
SELECT
    column_name,
    histogram->>'$."number-of-buckets-specified"' as buckets
FROM information_schema.column_statistics
WHERE table_name = 'users';
```

## Cardinality Estimation

### Basic Formulas

```
┌─────────────────────────────────────────────────────────────┐
│                Cardinality Estimation Formulas               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  BASE TABLE SCAN:                                            │
│    Cardinality = n_rows × selectivity(predicates)           │
│                                                              │
│  EQUALITY PREDICATE (column = value):                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ If value in MCV list:                                │    │
│  │   selectivity = mcv_frequency[value]                │    │
│  │ Else:                                                │    │
│  │   selectivity = (1 - sum(mcv_freq)) / n_distinct    │    │
│  │                                                      │    │
│  │ Example: status = 'active'                          │    │
│  │   MCV: 'active' → 0.70                              │    │
│  │   Selectivity = 0.70                                 │    │
│  │   Cardinality = 1,000,000 × 0.70 = 700,000          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  RANGE PREDICATE (column > value):                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Use histogram bounds to estimate fraction           │    │
│  │                                                      │    │
│  │ Example: age > 30, histogram bounds [0,20,40,60,80] │    │
│  │   Fraction above 30 in bucket [20,40]: (40-30)/20=0.5│    │
│  │   Plus buckets [40,60], [60,80], [80,100]           │    │
│  │   Total selectivity ≈ 0.5×0.25 + 3×0.25 = 0.875     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COMPOUND PREDICATES:                                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ AND: sel(A AND B) = sel(A) × sel(B)                 │    │
│  │      (assumes independence - often wrong!)          │    │
│  │                                                      │    │
│  │ OR:  sel(A OR B) = sel(A) + sel(B) - sel(A)×sel(B) │    │
│  │                                                      │    │
│  │ NOT: sel(NOT A) = 1 - sel(A)                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Join Cardinality

```
┌─────────────────────────────────────────────────────────────┐
│                  Join Cardinality Estimation                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Basic Formula:                                              │
│  |R ⋈ S| = |R| × |S| / max(n_distinct(R.a), n_distinct(S.b))│
│                                                              │
│  Example: orders ⋈ customers ON customer_id                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ |orders| = 1,000,000                                │    │
│  │ |customers| = 100,000                               │    │
│  │ n_distinct(orders.customer_id) = 80,000             │    │
│  │ n_distinct(customers.id) = 100,000                  │    │
│  │                                                      │    │
│  │ Join cardinality = 1,000,000 × 100,000 / 100,000   │    │
│  │                  = 1,000,000 rows                    │    │
│  │                                                      │    │
│  │ (Makes sense: each order matches one customer)      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  FK-PK Join (special case):                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ |R ⋈ S| = |R| when R.fk → S.pk                     │    │
│  │                                                      │    │
│  │ Every FK matches exactly one PK                     │    │
│  │ Result size equals size of FK table                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Multi-way Join:                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ A ⋈ B ⋈ C                                          │    │
│  │                                                      │    │
│  │ Estimate incrementally:                              │    │
│  │ |A ⋈ B| = ... (as above)                           │    │
│  │ |(A ⋈ B) ⋈ C| = |A ⋈ B| × |C| / n_distinct(...)  │    │
│  │                                                      │    │
│  │ ⚠️ Errors compound with each join!                 │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Common Estimation Errors

### Correlation Issues

```
┌─────────────────────────────────────────────────────────────┐
│            Correlated Columns Problem                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Query: WHERE city = 'San Francisco' AND state = 'CA'       │
│                                                              │
│  Independent assumption:                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ sel(city='SF') = 1/1000 cities = 0.001              │    │
│  │ sel(state='CA') = 1/50 states = 0.02                │    │
│  │ Combined: 0.001 × 0.02 = 0.00002                    │    │
│  │ Estimated rows: 1M × 0.00002 = 20 rows              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Reality:                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ City implies state (functional dependency)          │    │
│  │ sel(city='SF' AND state='CA') = sel(city='SF')     │    │
│  │ = 0.001                                              │    │
│  │ Actual rows: 1M × 0.001 = 1000 rows                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Error: 50x underestimate!                                  │
│  Result: May choose nested loop when hash join better       │
│                                                              │
│  Solutions:                                                  │
│  • Multi-column statistics (CREATE STATISTICS)             │
│  • Extended statistics for functional dependencies          │
│  • Adaptive query execution                                  │
└─────────────────────────────────────────────────────────────┘
```

### PostgreSQL Extended Statistics

```sql
-- Create multi-column statistics for correlated columns
CREATE STATISTICS city_state_stats (dependencies, ndistinct, mcv)
ON city, state FROM locations;

ANALYZE locations;

-- View the statistics
SELECT stxname, stxkeys, stxkind
FROM pg_statistic_ext
WHERE stxname = 'city_state_stats';

-- Check if planner uses them
EXPLAIN ANALYZE
SELECT * FROM locations
WHERE city = 'San Francisco' AND state = 'CA';

-- For functional dependencies
CREATE STATISTICS zip_city_stats (dependencies)
ON zip_code, city FROM addresses;
```

### Skewed Data

```
┌─────────────────────────────────────────────────────────────┐
│                    Data Skew Problems                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Example: E-commerce product table                          │
│                                                              │
│  Product orders distribution:                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Product A (bestseller): 500,000 orders (50%)        │    │
│  │ Product B (popular):    200,000 orders (20%)        │    │
│  │ Product C (average):     50,000 orders (5%)         │    │
│  │ 10,000 other products:  250,000 orders (25%)        │    │
│  │ = 25 orders each on average                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Query: WHERE product_id = 'A'                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Using uniform distribution:                          │    │
│  │   Estimate: 1M / 10,003 products = 100 rows         │    │
│  │   Actual: 500,000 rows                               │    │
│  │   Error: 5000x underestimate!                        │    │
│  │                                                      │    │
│  │ Using MCV (if 'A' is in list):                      │    │
│  │   Estimate: 1M × 0.50 = 500,000 rows               │    │
│  │   Accurate!                                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Solution: Increase statistics target for skewed columns    │
│  ALTER TABLE orders ALTER COLUMN product_id SET STATISTICS 500;
└─────────────────────────────────────────────────────────────┘
```

## Monitoring Statistics Quality

### PostgreSQL

```sql
-- Check estimation accuracy
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'pending';
-- Compare "rows=N" (estimate) with "actual rows=M"

-- Find tables with stale statistics
SELECT
    schemaname,
    relname,
    n_live_tup,
    n_dead_tup,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE n_dead_tup > n_live_tup * 0.1  -- >10% dead tuples
   OR last_analyze < NOW() - INTERVAL '7 days';

-- Check for large estimation errors in recent queries
SELECT
    query,
    calls,
    rows / calls as avg_rows,
    mean_time
FROM pg_stat_statements
WHERE rows / calls > 10 * (SELECT AVG(rows/calls) FROM pg_stat_statements)
ORDER BY mean_time DESC
LIMIT 10;
```

### Auto-Vacuum and Auto-Analyze

```sql
-- View autovacuum settings
SHOW autovacuum_analyze_threshold;      -- Default: 50
SHOW autovacuum_analyze_scale_factor;   -- Default: 0.1 (10%)

-- Analyze triggered when:
-- dead_tuples > threshold + scale_factor × n_live_tuples

-- For a table with 100,000 rows:
-- Analyze after: 50 + 0.1 × 100,000 = 10,050 changes

-- Adjust for critical tables
ALTER TABLE orders SET (
    autovacuum_analyze_threshold = 100,
    autovacuum_analyze_scale_factor = 0.02  -- 2%
);
```

## Key Takeaways

1. **Statistics drive optimizer decisions** - Stale stats = bad plans
2. **Histograms handle range queries** - Equi-height preferred for skew
3. **MCV lists essential for popular values** - Increase target for skewed data
4. **Independence assumption often fails** - Use extended statistics
5. **Run ANALYZE after bulk changes** - Don't wait for autovacuum
6. **Monitor estimation accuracy** - EXPLAIN ANALYZE shows reality vs estimate
