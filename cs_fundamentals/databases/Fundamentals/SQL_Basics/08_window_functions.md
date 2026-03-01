# Window Functions

## 1. Introduction

**Window functions** perform calculations across a set of table rows that are related to the current row. Unlike aggregate functions, window functions don't collapse rows - each row retains its identity.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WINDOW FUNCTIONS vs AGGREGATES                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   AGGREGATE (GROUP BY)                  WINDOW FUNCTION                      │
│   ────────────────────                  ───────────────                      │
│   Collapses rows into groups            Keeps all rows                       │
│   One output row per group              One output row per input row         │
│   Uses GROUP BY                         Uses OVER clause                     │
│                                                                              │
│   SELECT dept, AVG(salary)              SELECT name, salary,                 │
│   FROM employees                               AVG(salary) OVER()           │
│   GROUP BY dept;                        FROM employees;                      │
│                                                                              │
│   Result: 3 rows (one per dept)         Result: All employee rows            │
│   ┌──────┬────────┐                     ┌───────┬────────┬─────────┐        │
│   │ dept │  avg   │                     │ name  │ salary │   avg   │        │
│   │ Eng  │ 75000  │                     │ Alice │ 80000  │  70000  │        │
│   │ Sales│ 65000  │                     │ Bob   │ 70000  │  70000  │        │
│   │ HR   │ 60000  │                     │ Carol │ 60000  │  70000  │        │
│   └──────┴────────┘                     └───────┴────────┴─────────┘        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Window Function Syntax

```sql
function_name(expression) OVER (
    [PARTITION BY partition_expression, ...]
    [ORDER BY sort_expression [ASC | DESC], ...]
    [frame_clause]
)
```

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       WINDOW FUNCTION ANATOMY                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ROW_NUMBER() OVER (                                                        │
│       PARTITION BY department     ← Divide rows into groups                 │
│       ORDER BY salary DESC        ← Sort within each partition              │
│       ROWS BETWEEN ...            ← Define frame of rows                    │
│   )                                                                          │
│                                                                              │
│   PARTITION BY: Creates independent groups (like GROUP BY but keeps rows)  │
│   ORDER BY: Determines row order within partition                           │
│   Frame: Defines which rows relative to current row to include              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Sample Data

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary INT,
    hire_date DATE
);

INSERT INTO employees VALUES
    (1, 'Alice', 'Engineering', 80000, '2020-01-15'),
    (2, 'Bob', 'Engineering', 75000, '2019-06-01'),
    (3, 'Carol', 'Engineering', 90000, '2018-03-20'),
    (4, 'Dave', 'Sales', 70000, '2021-02-10'),
    (5, 'Eve', 'Sales', 65000, '2020-08-15'),
    (6, 'Frank', 'Sales', 72000, '2019-11-30'),
    (7, 'Grace', 'HR', 55000, '2022-01-05'),
    (8, 'Henry', 'HR', 58000, '2020-09-12');
```

---

## 4. Ranking Functions

### 4.1 ROW_NUMBER()

Assigns unique sequential integers to rows.

```sql
-- Row number within entire result
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- Result:
-- | name  | department  | salary | rank |
-- |-------|-------------|--------|------|
-- | Carol | Engineering | 90000  | 1    |
-- | Alice | Engineering | 80000  | 2    |
-- | Bob   | Engineering | 75000  | 3    |
-- | Frank | Sales       | 72000  | 4    |
-- ...

-- Row number within each department
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS dept_rank
FROM employees;

-- Result:
-- | name  | department  | salary | dept_rank |
-- |-------|-------------|--------|-----------|
-- | Carol | Engineering | 90000  | 1         |
-- | Alice | Engineering | 80000  | 2         |
-- | Bob   | Engineering | 75000  | 3         |
-- | Frank | Sales       | 72000  | 1         |
-- | Dave  | Sales       | 70000  | 2         |
-- ...

-- Get top N per group
WITH ranked AS (
    SELECT *,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT name, department, salary
FROM ranked
WHERE rn <= 2;  -- Top 2 per department
```

### 4.2 RANK() and DENSE_RANK()

Handle ties differently than ROW_NUMBER().

```sql
-- Add some tie data
INSERT INTO employees VALUES
    (9, 'Ivan', 'Engineering', 75000, '2021-05-01');  -- Same salary as Bob

SELECT
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees
ORDER BY salary DESC;

-- Result:
-- | name  | salary | row_num | rank | dense_rank |
-- |-------|--------|---------|------|------------|
-- | Carol | 90000  | 1       | 1    | 1          |
-- | Alice | 80000  | 2       | 2    | 2          |
-- | Bob   | 75000  | 3       | 3    | 3          |
-- | Ivan  | 75000  | 4       | 3    | 3          | ← Ties
-- | Frank | 72000  | 5       | 5    | 4          | ← RANK skips to 5, DENSE_RANK continues
-- | Dave  | 70000  | 6       | 6    | 5          |

-- ROW_NUMBER: Always unique (arbitrary tiebreaker)
-- RANK: Same rank for ties, skips next ranks
-- DENSE_RANK: Same rank for ties, doesn't skip
```

### 4.3 NTILE()

Divides rows into N approximately equal groups.

```sql
-- Divide employees into 4 salary quartiles
SELECT
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;

-- Result:
-- | name  | salary | quartile |
-- |-------|--------|----------|
-- | Carol | 90000  | 1        | ← Top 25%
-- | Alice | 80000  | 1        |
-- | Bob   | 75000  | 2        |
-- | Ivan  | 75000  | 2        |
-- | Frank | 72000  | 3        |
-- | Dave  | 70000  | 3        |
-- | Eve   | 65000  | 4        | ← Bottom 25%
-- | Henry | 58000  | 4        |
-- | Grace | 55000  | 4        |

-- Percentile rank
SELECT
    name,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS percent_rank,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_dist
FROM employees;
```

---

## 5. Aggregate Window Functions

Use standard aggregates as window functions.

```sql
-- Running totals and moving averages
SELECT
    name,
    department,
    salary,
    SUM(salary) OVER () AS total_company_salary,
    SUM(salary) OVER (PARTITION BY department) AS dept_salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    COUNT(*) OVER (PARTITION BY department) AS dept_count
FROM employees;

-- Result:
-- | name  | department  | salary | total  | dept   | dept_avg | dept_count |
-- |-------|-------------|--------|--------|--------|----------|------------|
-- | Alice | Engineering | 80000  | 640000 | 245000 | 81666    | 3          |
-- | Bob   | Engineering | 75000  | 640000 | 245000 | 81666    | 3          |
-- | Carol | Engineering | 90000  | 640000 | 245000 | 81666    | 3          |
-- | Dave  | Sales       | 70000  | 640000 | 207000 | 69000    | 3          |
-- ...

-- Running total (cumulative sum)
SELECT
    name,
    hire_date,
    salary,
    SUM(salary) OVER (ORDER BY hire_date) AS running_total
FROM employees
ORDER BY hire_date;

-- Salary relative to department average
SELECT
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg,
    ROUND(salary * 100.0 / SUM(salary) OVER (PARTITION BY department), 2) AS pct_of_dept
FROM employees;
```

---

## 6. Value Functions

### 6.1 LAG() and LEAD()

Access values from previous/next rows.

```sql
-- Compare to previous row
SELECT
    name,
    hire_date,
    salary,
    LAG(salary) OVER (ORDER BY hire_date) AS prev_salary,
    salary - LAG(salary) OVER (ORDER BY hire_date) AS salary_diff
FROM employees
ORDER BY hire_date;

-- Result:
-- | name  | hire_date  | salary | prev_salary | salary_diff |
-- |-------|------------|--------|-------------|-------------|
-- | Carol | 2018-03-20 | 90000  | NULL        | NULL        |
-- | Bob   | 2019-06-01 | 75000  | 90000       | -15000      |
-- | Frank | 2019-11-30 | 72000  | 75000       | -3000       |
-- | Alice | 2020-01-15 | 80000  | 72000       | 8000        |
-- ...

-- Look ahead with LEAD
SELECT
    name,
    salary,
    LEAD(salary) OVER (ORDER BY salary DESC) AS next_lower_salary,
    salary - LEAD(salary) OVER (ORDER BY salary DESC) AS gap_to_next
FROM employees
ORDER BY salary DESC;

-- With offset and default
SELECT
    name,
    salary,
    LAG(salary, 2, 0) OVER (ORDER BY hire_date) AS salary_2_hires_ago
FROM employees;

-- Month-over-month comparison
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(total) AS revenue,
    LAG(SUM(total)) OVER (ORDER BY DATE_TRUNC('month', order_date)) AS prev_month,
    SUM(total) - LAG(SUM(total)) OVER (ORDER BY DATE_TRUNC('month', order_date)) AS mom_change
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

### 6.2 FIRST_VALUE() and LAST_VALUE()

Get first/last value in the window frame.

```sql
-- First and last in partition
SELECT
    name,
    department,
    salary,
    FIRST_VALUE(name) OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS highest_paid,
    LAST_VALUE(name) OVER (
        PARTITION BY department
        ORDER BY salary DESC
        RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid
FROM employees;

-- Note: LAST_VALUE requires explicit frame for expected behavior
-- Default frame is RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

### 6.3 NTH_VALUE()

Get the Nth value in the window.

```sql
-- Second highest salary per department
SELECT
    name,
    department,
    salary,
    NTH_VALUE(name, 2) OVER (
        PARTITION BY department
        ORDER BY salary DESC
        RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_highest_paid
FROM employees;
```

---

## 7. Frame Specification

Define which rows are included in the calculation relative to current row.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FRAME SPECIFICATION                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ROWS BETWEEN start_bound AND end_bound                                    │
│   RANGE BETWEEN start_bound AND end_bound                                   │
│                                                                              │
│   Bounds:                                                                    │
│   • UNBOUNDED PRECEDING   - First row of partition                          │
│   • n PRECEDING           - n rows before current                           │
│   • CURRENT ROW           - The current row                                 │
│   • n FOLLOWING           - n rows after current                            │
│   • UNBOUNDED FOLLOWING   - Last row of partition                           │
│                                                                              │
│   Example frames:                                                            │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ Row 1: ████                                                          │   │
│   │ Row 2: ████████                                                      │   │
│   │ Row 3: ████████████  ← CURRENT ROW                                   │   │
│   │ Row 4:     ████████████                                              │   │
│   │ Row 5:         ████████████                                          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│   ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING (5-row moving window)           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- Different frame specifications
SELECT
    name,
    salary,
    -- Running total (all rows up to current)
    SUM(salary) OVER (
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,

    -- 3-row moving average (current + 1 before + 1 after)
    AVG(salary) OVER (
        ORDER BY hire_date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS moving_avg_3,

    -- Future sum (current row to end)
    SUM(salary) OVER (
        ORDER BY hire_date
        ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
    ) AS future_total
FROM employees
ORDER BY hire_date;

-- ROWS vs RANGE
-- ROWS: Physical rows
-- RANGE: Logical value range (handles duplicates differently)

SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_rows,
    SUM(amount) OVER (
        ORDER BY order_date
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_range  -- Includes all rows with same date
FROM orders;
```

---

## 8. Practical Examples

### 8.1 Year-over-Year Comparison

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(total) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY month) AS revenue_last_year,
    revenue - LAG(revenue, 12) OVER (ORDER BY month) AS yoy_change,
    ROUND(
        (revenue - LAG(revenue, 12) OVER (ORDER BY month)) * 100.0 /
        NULLIF(LAG(revenue, 12) OVER (ORDER BY month), 0),
        2
    ) AS yoy_pct_change
FROM monthly_sales
ORDER BY month;
```

### 8.2 Gap Analysis

```sql
-- Find gaps in sequential IDs
WITH numbered AS (
    SELECT
        id,
        ROW_NUMBER() OVER (ORDER BY id) AS expected_id
    FROM orders
)
SELECT
    id AS actual_id,
    expected_id,
    id - expected_id AS gap_size
FROM numbered
WHERE id != expected_id;

-- Time between events
SELECT
    user_id,
    event_time,
    LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS prev_event,
    event_time - LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS time_since_last
FROM user_events;
```

### 8.3 Sessionization

```sql
-- Group events into sessions (30-minute gap = new session)
WITH events_with_gap AS (
    SELECT
        user_id,
        event_time,
        EXTRACT(EPOCH FROM (
            event_time - LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time)
        )) / 60 AS minutes_since_last
    FROM user_events
),
session_starts AS (
    SELECT
        *,
        CASE
            WHEN minutes_since_last IS NULL OR minutes_since_last > 30
            THEN 1
            ELSE 0
        END AS is_session_start
    FROM events_with_gap
)
SELECT
    user_id,
    event_time,
    SUM(is_session_start) OVER (
        PARTITION BY user_id
        ORDER BY event_time
    ) AS session_id
FROM session_starts;
```

### 8.4 Percentiles and Distribution

```sql
-- Salary percentiles
SELECT
    name,
    department,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS percentile,
    NTILE(100) OVER (ORDER BY salary) AS percentile_bucket
FROM employees;

-- Salary distribution by department
SELECT
    department,
    MIN(salary) AS min_salary,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) AS p25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY salary) AS median,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) AS p75,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department;
```

---

## 9. Code Examples Across Languages

### Python (pandas)

```python
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Carol', 'Dave', 'Eve'],
    'department': ['Eng', 'Eng', 'Eng', 'Sales', 'Sales'],
    'salary': [80000, 75000, 90000, 70000, 65000],
    'hire_date': pd.to_datetime(['2020-01-15', '2019-06-01', '2018-03-20',
                                  '2021-02-10', '2020-08-15'])
})

# Ranking
df['rank'] = df['salary'].rank(method='min', ascending=False)
df['dept_rank'] = df.groupby('department')['salary'].rank(method='min', ascending=False)

# Aggregate window functions
df['dept_avg'] = df.groupby('department')['salary'].transform('mean')
df['pct_of_dept'] = df['salary'] / df['dept_avg']

# Lag/Lead
df = df.sort_values('hire_date')
df['prev_salary'] = df['salary'].shift(1)
df['next_salary'] = df['salary'].shift(-1)

# Running total
df['running_total'] = df['salary'].cumsum()

# Moving average (3-period)
df['moving_avg'] = df['salary'].rolling(window=3, center=True).mean()
```

### SQL Alchemy (Python)

```python
from sqlalchemy import func, over
from sqlalchemy.orm import Session

# Window function in SQLAlchemy
query = session.query(
    Employee.name,
    Employee.department,
    Employee.salary,
    func.row_number().over(
        partition_by=Employee.department,
        order_by=Employee.salary.desc()
    ).label('dept_rank'),
    func.avg(Employee.salary).over(
        partition_by=Employee.department
    ).label('dept_avg')
)

for row in query:
    print(f"{row.name}: Rank {row.dept_rank} in {row.department}")
```

---

## 10. Summary

| Function Type | Functions | Use Case |
|--------------|-----------|----------|
| **Ranking** | ROW_NUMBER, RANK, DENSE_RANK, NTILE | Ordering, top-N, percentiles |
| **Aggregate** | SUM, AVG, COUNT, MIN, MAX | Running totals, moving averages |
| **Value** | LAG, LEAD, FIRST_VALUE, LAST_VALUE, NTH_VALUE | Comparisons, accessing other rows |

**Key Points:**
- Window functions don't collapse rows like GROUP BY
- PARTITION BY creates groups; ORDER BY determines row order
- Frame clause controls which rows are included in calculation
- Essential for time-series analysis, rankings, and running calculations
- Usually more efficient than correlated subqueries
