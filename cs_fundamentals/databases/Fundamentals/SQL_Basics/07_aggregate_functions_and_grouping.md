# Aggregate Functions and Grouping

## 1. Introduction

**Aggregate functions** perform calculations on sets of rows and return a single result. Combined with **GROUP BY**, they enable powerful data summarization.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AGGREGATE FUNCTIONS OVERVIEW                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Basic Aggregates:                                                         │
│   • COUNT()  - Number of rows                                               │
│   • SUM()    - Total of values                                              │
│   • AVG()    - Average of values                                            │
│   • MIN()    - Minimum value                                                │
│   • MAX()    - Maximum value                                                │
│                                                                              │
│   Statistical Aggregates:                                                   │
│   • STDDEV() - Standard deviation                                           │
│   • VARIANCE() - Variance                                                   │
│   • PERCENTILE_CONT() - Percentile                                          │
│                                                                              │
│   Grouping Clauses:                                                         │
│   • GROUP BY   - Group rows for aggregation                                 │
│   • HAVING     - Filter groups (like WHERE for groups)                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Basic Aggregate Functions

### 2.1 COUNT

```sql
-- Count all rows
SELECT COUNT(*) FROM employees;

-- Count non-NULL values in a column
SELECT COUNT(manager_id) FROM employees;  -- Excludes NULLs

-- Count distinct values
SELECT COUNT(DISTINCT department_id) FROM employees;

-- Count with condition (PostgreSQL, newer MySQL)
SELECT COUNT(*) FILTER (WHERE salary > 50000) AS high_earners
FROM employees;

-- Count with CASE (all databases)
SELECT
    COUNT(CASE WHEN salary > 50000 THEN 1 END) AS high_earners,
    COUNT(CASE WHEN salary <= 50000 THEN 1 END) AS regular_earners
FROM employees;
```

### 2.2 SUM

```sql
-- Total salary
SELECT SUM(salary) AS total_salary FROM employees;

-- Sum with condition
SELECT SUM(salary) FILTER (WHERE department_id = 10) AS dept_10_total
FROM employees;

-- Sum distinct values (unusual but possible)
SELECT SUM(DISTINCT salary) FROM employees;  -- Each unique salary counted once

-- Sum of expression
SELECT SUM(quantity * unit_price) AS total_revenue FROM order_items;
```

### 2.3 AVG

```sql
-- Average salary
SELECT AVG(salary) AS avg_salary FROM employees;

-- Average ignores NULLs
-- To treat NULL as 0:
SELECT AVG(COALESCE(bonus, 0)) AS avg_bonus FROM employees;

-- Average of distinct values
SELECT AVG(DISTINCT salary) FROM employees;

-- Weighted average
SELECT
    SUM(salary * performance_score) / SUM(performance_score) AS weighted_avg
FROM employees
WHERE performance_score IS NOT NULL;
```

### 2.4 MIN and MAX

```sql
-- Minimum and maximum
SELECT
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary,
    MAX(salary) - MIN(salary) AS salary_range
FROM employees;

-- Works with dates
SELECT
    MIN(hire_date) AS first_hire,
    MAX(hire_date) AS last_hire
FROM employees;

-- Works with strings (alphabetically)
SELECT
    MIN(last_name) AS first_alphabetically,
    MAX(last_name) AS last_alphabetically
FROM employees;
```

---

## 3. GROUP BY Clause

### 3.1 Basic Grouping

```sql
-- Group by single column
SELECT
    department_id,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;

-- Group by multiple columns
SELECT
    department_id,
    job_title,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id, job_title;
```

### 3.2 GROUP BY Rules

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       GROUP BY RULES                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   RULE 1: Every non-aggregate column in SELECT must be in GROUP BY         │
│                                                                              │
│   CORRECT:                                                                   │
│   SELECT department_id, COUNT(*) FROM employees GROUP BY department_id;     │
│                                                                              │
│   WRONG:                                                                     │
│   SELECT department_id, first_name, COUNT(*) FROM employees                 │
│   GROUP BY department_id;  -- first_name not in GROUP BY!                   │
│                                                                              │
│   RULE 2: GROUP BY can include columns not in SELECT                        │
│   SELECT COUNT(*) FROM employees GROUP BY department_id;  -- Valid          │
│                                                                              │
│   RULE 3: GROUP BY is evaluated after WHERE, before SELECT                  │
│   FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 GROUP BY with Expressions

```sql
-- Group by expression
SELECT
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    COUNT(*) AS employees_hired
FROM employees
GROUP BY EXTRACT(YEAR FROM hire_date)
ORDER BY hire_year;

-- Group by calculated column
SELECT
    CASE
        WHEN salary < 50000 THEN 'Entry'
        WHEN salary < 80000 THEN 'Mid'
        ELSE 'Senior'
    END AS salary_band,
    COUNT(*) AS emp_count
FROM employees
GROUP BY
    CASE
        WHEN salary < 50000 THEN 'Entry'
        WHEN salary < 80000 THEN 'Mid'
        ELSE 'Senior'
    END;

-- Using alias in GROUP BY (PostgreSQL)
SELECT
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    COUNT(*)
FROM employees
GROUP BY hire_year;  -- PostgreSQL allows this
```

---

## 4. HAVING Clause

HAVING filters groups after aggregation (WHERE filters rows before).

### 4.1 Basic HAVING

```sql
-- Departments with more than 5 employees
SELECT
    department_id,
    COUNT(*) AS emp_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;

-- Departments with high average salary
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 60000;

-- Multiple HAVING conditions
SELECT
    department_id,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) >= 3 AND AVG(salary) > 50000;
```

### 4.2 HAVING vs WHERE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       WHERE vs HAVING                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   WHERE                          HAVING                                     │
│   ────────────────────           ─────────────────────────                  │
│   Filters ROWS                   Filters GROUPS                             │
│   Before GROUP BY                After GROUP BY                             │
│   Can't use aggregates           Can use aggregates                         │
│   Works without GROUP BY         Requires GROUP BY (usually)                │
│                                                                              │
│   Example - Filter employees, then group:                                   │
│   SELECT department_id, AVG(salary)                                         │
│   FROM employees                                                            │
│   WHERE hire_date > '2020-01-01'  -- Filter rows first                     │
│   GROUP BY department_id                                                    │
│   HAVING AVG(salary) > 50000;     -- Then filter groups                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Statistical Aggregate Functions

### 5.1 Standard Deviation and Variance

```sql
-- Population vs Sample statistics
SELECT
    AVG(salary) AS mean,
    STDDEV_POP(salary) AS std_population,    -- Population std dev
    STDDEV_SAMP(salary) AS std_sample,       -- Sample std dev (alias: STDDEV)
    VAR_POP(salary) AS var_population,       -- Population variance
    VAR_SAMP(salary) AS var_sample           -- Sample variance (alias: VARIANCE)
FROM employees;

-- Coefficient of variation
SELECT
    AVG(salary) AS mean,
    STDDEV(salary) AS std_dev,
    STDDEV(salary) / AVG(salary) * 100 AS cv_percent
FROM employees;
```

### 5.2 Percentiles (PostgreSQL)

```sql
-- Median (50th percentile)
SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees;

-- Multiple percentiles
SELECT
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) AS p25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY salary) AS p50,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) AS p75,
    PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY salary) AS p90
FROM employees;

-- Percentile by group
SELECT
    department_id,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees
GROUP BY department_id;

-- Discrete percentile (returns actual value from dataset)
SELECT
    PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees;
```

### 5.3 Mode (PostgreSQL)

```sql
-- Most common value
SELECT
    MODE() WITHIN GROUP (ORDER BY department_id) AS most_common_dept
FROM employees;
```

---

## 6. String Aggregates

### 6.1 STRING_AGG / GROUP_CONCAT

```sql
-- PostgreSQL: STRING_AGG
SELECT
    department_id,
    STRING_AGG(first_name, ', ' ORDER BY first_name) AS employee_names
FROM employees
GROUP BY department_id;

-- MySQL: GROUP_CONCAT
SELECT
    department_id,
    GROUP_CONCAT(first_name ORDER BY first_name SEPARATOR ', ') AS employee_names
FROM employees
GROUP BY department_id;

-- With DISTINCT
SELECT
    department_id,
    STRING_AGG(DISTINCT job_title, ', ') AS job_titles
FROM employees
GROUP BY department_id;
```

### 6.2 Array Aggregation (PostgreSQL)

```sql
-- Collect values into array
SELECT
    department_id,
    ARRAY_AGG(first_name ORDER BY first_name) AS employee_names,
    ARRAY_AGG(DISTINCT job_title) AS job_titles
FROM employees
GROUP BY department_id;

-- JSON aggregation
SELECT
    department_id,
    JSON_AGG(ROW_TO_JSON(e.*)) AS employees_json
FROM employees e
GROUP BY department_id;
```

---

## 7. Boolean Aggregates

```sql
-- PostgreSQL
SELECT
    department_id,
    BOOL_AND(active) AS all_active,        -- True if all are true
    BOOL_OR(on_probation) AS any_probation -- True if any is true
FROM employees
GROUP BY department_id;

-- All databases (using CASE)
SELECT
    department_id,
    CASE WHEN COUNT(*) = SUM(CASE WHEN active THEN 1 ELSE 0 END)
         THEN TRUE ELSE FALSE END AS all_active,
    CASE WHEN SUM(CASE WHEN on_probation THEN 1 ELSE 0 END) > 0
         THEN TRUE ELSE FALSE END AS any_probation
FROM employees
GROUP BY department_id;
```

---

## 8. GROUPING SETS, CUBE, and ROLLUP

### 8.1 GROUPING SETS

```sql
-- Multiple groupings in one query
SELECT
    department_id,
    job_title,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY GROUPING SETS (
    (department_id, job_title),  -- By department and job
    (department_id),              -- By department only
    (job_title),                  -- By job only
    ()                            -- Grand total
);
```

### 8.2 ROLLUP

```sql
-- Hierarchical totals
SELECT
    COALESCE(department_id::TEXT, 'ALL DEPTS') AS department,
    COALESCE(job_title, 'ALL JOBS') AS job,
    COUNT(*) AS emp_count,
    SUM(salary) AS total_salary
FROM employees
GROUP BY ROLLUP (department_id, job_title);

-- Result includes:
-- Each department + job combination
-- Each department subtotal (job_title = NULL)
-- Grand total (both = NULL)
```

### 8.3 CUBE

```sql
-- All possible combinations
SELECT
    department_id,
    job_title,
    hire_year,
    COUNT(*) AS emp_count
FROM employees
CROSS JOIN LATERAL (VALUES (EXTRACT(YEAR FROM hire_date))) AS t(hire_year)
GROUP BY CUBE (department_id, job_title, hire_year);

-- Generates all combinations:
-- (dept, job, year), (dept, job), (dept, year), (job, year)
-- (dept), (job), (year), ()
```

### 8.4 GROUPING Function

```sql
-- Identify which columns are NULL due to grouping
SELECT
    CASE WHEN GROUPING(department_id) = 1 THEN 'All Depts'
         ELSE department_id::TEXT END AS department,
    CASE WHEN GROUPING(job_title) = 1 THEN 'All Jobs'
         ELSE job_title END AS job,
    COUNT(*) AS emp_count,
    GROUPING(department_id) AS is_dept_total,
    GROUPING(job_title) AS is_job_total
FROM employees
GROUP BY ROLLUP (department_id, job_title);
```

---

## 9. Advanced Aggregation Patterns

### 9.1 Conditional Aggregation

```sql
-- Pivot-style aggregation
SELECT
    department_id,
    COUNT(*) AS total_employees,
    COUNT(CASE WHEN salary < 50000 THEN 1 END) AS low_salary,
    COUNT(CASE WHEN salary BETWEEN 50000 AND 80000 THEN 1 END) AS mid_salary,
    COUNT(CASE WHEN salary > 80000 THEN 1 END) AS high_salary,
    SUM(CASE WHEN gender = 'M' THEN salary END) AS male_salary_total,
    SUM(CASE WHEN gender = 'F' THEN salary END) AS female_salary_total
FROM employees
GROUP BY department_id;

-- PostgreSQL FILTER syntax (cleaner)
SELECT
    department_id,
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE salary > 80000) AS high_earners,
    AVG(salary) FILTER (WHERE hire_date > '2020-01-01') AS new_hire_avg
FROM employees
GROUP BY department_id;
```

### 9.2 Running Totals and Cumulative Aggregates

```sql
-- Running total using window function
SELECT
    order_date,
    total,
    SUM(total) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Running average
SELECT
    order_date,
    total,
    AVG(total) OVER (ORDER BY order_date
                     ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_avg
FROM orders;
```

### 9.3 Top N Per Group

```sql
-- Top 3 earners per department
WITH ranked AS (
    SELECT
        department_id,
        first_name,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT * FROM ranked WHERE rn <= 3;

-- Aggregated with subquery
SELECT
    department_id,
    STRING_AGG(first_name, ', ' ORDER BY salary DESC) AS top_earners
FROM (
    SELECT
        department_id,
        first_name,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employees
) ranked
WHERE rn <= 3
GROUP BY department_id;
```

### 9.4 Aggregate with NULL Handling

```sql
-- Count NULLs
SELECT
    COUNT(*) AS total_rows,
    COUNT(manager_id) AS has_manager,
    COUNT(*) - COUNT(manager_id) AS no_manager,
    SUM(CASE WHEN manager_id IS NULL THEN 1 ELSE 0 END) AS null_count
FROM employees;

-- Average with NULL as zero
SELECT
    AVG(COALESCE(bonus, 0)) AS avg_with_nulls_as_zero,
    AVG(bonus) AS avg_excluding_nulls
FROM employees;
```

---

## 10. Performance Considerations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 AGGREGATION PERFORMANCE TIPS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. INDEX ON GROUP BY COLUMNS                                              │
│      CREATE INDEX idx_dept ON employees(department_id);                     │
│      Allows index-only scan for GROUP BY                                    │
│                                                                              │
│   2. FILTER EARLY WITH WHERE                                                │
│      WHERE reduces rows before expensive GROUP BY                           │
│                                                                              │
│   3. AVOID SELECT * WITH GROUP BY                                           │
│      Only select needed columns                                             │
│                                                                              │
│   4. USE APPROXIMATE COUNTS FOR LARGE TABLES                                │
│      PostgreSQL: SELECT reltuples FROM pg_class WHERE relname = 'table';    │
│      HyperLogLog for distinct counts                                        │
│                                                                              │
│   5. CONSIDER MATERIALIZED VIEWS                                            │
│      Pre-compute common aggregations                                        │
│                                                                              │
│   6. PARTITION LARGE TABLES                                                 │
│      Aggregation can run on partitions in parallel                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. Summary

| Function | Description | Example |
|----------|-------------|---------|
| COUNT(*) | Count all rows | `COUNT(*)` |
| COUNT(col) | Count non-NULL values | `COUNT(manager_id)` |
| COUNT(DISTINCT) | Count unique values | `COUNT(DISTINCT dept_id)` |
| SUM | Total of values | `SUM(salary)` |
| AVG | Average | `AVG(salary)` |
| MIN/MAX | Minimum/Maximum | `MIN(hire_date)` |
| STDDEV | Standard deviation | `STDDEV(salary)` |
| STRING_AGG | Concatenate strings | `STRING_AGG(name, ', ')` |

| Clause | Purpose | Example |
|--------|---------|---------|
| GROUP BY | Group rows | `GROUP BY department_id` |
| HAVING | Filter groups | `HAVING COUNT(*) > 5` |
| ROLLUP | Hierarchical totals | `GROUP BY ROLLUP(a, b)` |
| CUBE | All combinations | `GROUP BY CUBE(a, b)` |
| GROUPING SETS | Custom groupings | `GROUP BY GROUPING SETS((a), (b))` |

**Key Points:**
- Every non-aggregate column in SELECT must be in GROUP BY
- WHERE filters rows before grouping, HAVING filters groups after
- Use FILTER clause (PostgreSQL) or CASE for conditional aggregation
- ROLLUP, CUBE, and GROUPING SETS enable multi-level aggregation
- Consider performance: index GROUP BY columns, filter early
