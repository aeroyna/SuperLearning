# DQL - Data Query Language

## 1. Introduction

**Data Query Language (DQL)** is the subset of SQL used to retrieve data from databases. The primary statement is `SELECT`, which is the most commonly used SQL command.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DQL OVERVIEW                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   DQL consists of SELECT and its clauses:                                   │
│                                                                              │
│   SELECT     - Columns to retrieve                                          │
│   FROM       - Tables to query                                              │
│   WHERE      - Row filter conditions                                        │
│   GROUP BY   - Group rows for aggregation                                   │
│   HAVING     - Filter groups                                                │
│   ORDER BY   - Sort results                                                 │
│   LIMIT      - Restrict number of rows                                      │
│                                                                              │
│   Execution Order (Logical):                                                │
│   FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Basic SELECT Syntax

### 2.1 Selecting Columns

```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary FROM employees;

-- Column aliases
SELECT
    first_name AS "First Name",
    last_name AS "Last Name",
    salary AS annual_salary
FROM employees;

-- Expressions in SELECT
SELECT
    first_name,
    last_name,
    salary,
    salary * 12 AS annual_salary,
    salary * 1.1 AS salary_with_raise
FROM employees;

-- DISTINCT - Remove duplicates
SELECT DISTINCT department_id FROM employees;
SELECT DISTINCT department_id, job_title FROM employees;
```

### 2.2 Literal Values and Concatenation

```sql
-- String concatenation (varies by database)
-- PostgreSQL/MySQL
SELECT first_name || ' ' || last_name AS full_name FROM employees;

-- SQL Server
SELECT first_name + ' ' + last_name AS full_name FROM employees;

-- MySQL CONCAT function
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- Including literal values
SELECT
    first_name,
    'works in department' AS text,
    department_id
FROM employees;
```

---

## 3. WHERE Clause - Filtering Rows

### 3.1 Comparison Operators

```sql
-- Basic comparisons
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE salary >= 50000;
SELECT * FROM employees WHERE salary < 50000;
SELECT * FROM employees WHERE salary <= 50000;
SELECT * FROM employees WHERE salary = 50000;
SELECT * FROM employees WHERE salary != 50000;  -- or <>

-- String comparisons (case sensitivity varies by database)
SELECT * FROM employees WHERE last_name = 'Smith';
SELECT * FROM employees WHERE last_name > 'M';  -- Alphabetical comparison
```

### 3.2 Logical Operators

```sql
-- AND - Both conditions must be true
SELECT * FROM employees
WHERE department_id = 10 AND salary > 50000;

-- OR - Either condition can be true
SELECT * FROM employees
WHERE department_id = 10 OR department_id = 20;

-- NOT - Negate condition
SELECT * FROM employees
WHERE NOT department_id = 10;

-- Complex conditions with parentheses
SELECT * FROM employees
WHERE (department_id = 10 OR department_id = 20)
  AND salary > 50000;
```

### 3.3 Special Operators

```sql
-- BETWEEN - Range (inclusive)
SELECT * FROM employees
WHERE salary BETWEEN 40000 AND 60000;
-- Equivalent to: salary >= 40000 AND salary <= 60000

-- IN - Match any value in list
SELECT * FROM employees
WHERE department_id IN (10, 20, 30);
-- Equivalent to: department_id = 10 OR department_id = 20 OR department_id = 30

-- NOT IN
SELECT * FROM employees
WHERE department_id NOT IN (10, 20);

-- LIKE - Pattern matching
SELECT * FROM employees WHERE last_name LIKE 'S%';      -- Starts with S
SELECT * FROM employees WHERE last_name LIKE '%son';    -- Ends with son
SELECT * FROM employees WHERE last_name LIKE '%mit%';   -- Contains mit
SELECT * FROM employees WHERE last_name LIKE '_mith';   -- 5 chars, ends with mith
SELECT * FROM employees WHERE email LIKE '%@gmail.com';

-- ILIKE - Case-insensitive LIKE (PostgreSQL)
SELECT * FROM employees WHERE last_name ILIKE 's%';

-- IS NULL / IS NOT NULL
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE manager_id IS NOT NULL;
-- Note: column = NULL never works, must use IS NULL
```

---

## 4. ORDER BY - Sorting Results

```sql
-- Ascending order (default)
SELECT * FROM employees ORDER BY last_name;
SELECT * FROM employees ORDER BY last_name ASC;

-- Descending order
SELECT * FROM employees ORDER BY salary DESC;

-- Multiple columns
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC;

-- Order by column position
SELECT first_name, last_name, salary
FROM employees
ORDER BY 3 DESC;  -- Order by third column (salary)

-- Order by expression
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary * 12 DESC;

-- Order by alias
SELECT first_name, last_name, salary * 12 AS annual
FROM employees
ORDER BY annual DESC;

-- NULLS FIRST / NULLS LAST (PostgreSQL, Oracle)
SELECT * FROM employees
ORDER BY manager_id NULLS FIRST;

SELECT * FROM employees
ORDER BY manager_id DESC NULLS LAST;
```

---

## 5. LIMIT and OFFSET - Pagination

### 5.1 Basic Limiting

```sql
-- MySQL, PostgreSQL, SQLite
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;

-- SQL Server
SELECT TOP 10 * FROM employees ORDER BY salary DESC;

-- Oracle (12c+)
SELECT * FROM employees ORDER BY salary DESC FETCH FIRST 10 ROWS ONLY;

-- Oracle (older)
SELECT * FROM (
    SELECT * FROM employees ORDER BY salary DESC
) WHERE ROWNUM <= 10;
```

### 5.2 Pagination with OFFSET

```sql
-- Page 1: rows 1-10
SELECT * FROM employees
ORDER BY employee_id
LIMIT 10 OFFSET 0;

-- Page 2: rows 11-20
SELECT * FROM employees
ORDER BY employee_id
LIMIT 10 OFFSET 10;

-- Page 3: rows 21-30
SELECT * FROM employees
ORDER BY employee_id
LIMIT 10 OFFSET 20;

-- Alternative syntax (MySQL)
SELECT * FROM employees
ORDER BY employee_id
LIMIT 10, 10;  -- LIMIT offset, count

-- SQL Server (2012+)
SELECT * FROM employees
ORDER BY employee_id
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;
```

### 5.3 Pagination Pattern

```sql
-- Generic pagination formula:
-- OFFSET = (page_number - 1) * page_size
-- LIMIT = page_size

-- Example: 25 items per page
-- Page 1: OFFSET 0, LIMIT 25
-- Page 2: OFFSET 25, LIMIT 25
-- Page 3: OFFSET 50, LIMIT 25

-- With total count for UI
SELECT
    (SELECT COUNT(*) FROM employees) AS total_count,
    e.*
FROM employees e
ORDER BY employee_id
LIMIT 25 OFFSET 50;
```

---

## 6. Query Execution Order

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SQL QUERY EXECUTION ORDER                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Written Order:          Execution Order:                                  │
│                                                                              │
│   SELECT columns      1.  FROM table                                        │
│   FROM table          2.  WHERE conditions (filter rows)                    │
│   WHERE conditions    3.  GROUP BY columns                                  │
│   GROUP BY columns    4.  HAVING conditions (filter groups)                 │
│   HAVING conditions   5.  SELECT columns (evaluate expressions)             │
│   ORDER BY columns    6.  DISTINCT (remove duplicates)                      │
│   LIMIT count         7.  ORDER BY columns                                  │
│                       8.  LIMIT/OFFSET                                      │
│                                                                              │
│   This is why you can't use column aliases in WHERE:                        │
│   SELECT salary * 12 AS annual FROM employees WHERE annual > 100000; ✗     │
│   (annual doesn't exist yet when WHERE is evaluated)                        │
│                                                                              │
│   But you CAN use aliases in ORDER BY:                                      │
│   SELECT salary * 12 AS annual FROM employees ORDER BY annual; ✓           │
│   (SELECT is evaluated before ORDER BY)                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. CASE Expressions

### 7.1 Simple CASE

```sql
-- Simple CASE (compares to specific values)
SELECT
    first_name,
    department_id,
    CASE department_id
        WHEN 10 THEN 'Engineering'
        WHEN 20 THEN 'Sales'
        WHEN 30 THEN 'Marketing'
        ELSE 'Other'
    END AS department_name
FROM employees;
```

### 7.2 Searched CASE

```sql
-- Searched CASE (evaluates conditions)
SELECT
    first_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'Executive'
        WHEN salary >= 70000 THEN 'Senior'
        WHEN salary >= 50000 THEN 'Mid-Level'
        WHEN salary >= 30000 THEN 'Junior'
        ELSE 'Entry'
    END AS level
FROM employees;

-- CASE in ORDER BY
SELECT * FROM employees
ORDER BY
    CASE
        WHEN department_id = 10 THEN 1
        WHEN department_id = 20 THEN 2
        ELSE 3
    END;

-- CASE for conditional aggregation
SELECT
    COUNT(*) AS total_employees,
    COUNT(CASE WHEN salary >= 50000 THEN 1 END) AS high_earners,
    COUNT(CASE WHEN salary < 50000 THEN 1 END) AS regular_earners
FROM employees;
```

---

## 8. NULL Handling

```sql
-- COALESCE - Return first non-null value
SELECT
    first_name,
    COALESCE(phone, mobile, 'No Phone') AS contact_number
FROM employees;

-- NULLIF - Return NULL if values are equal
SELECT NULLIF(bonus, 0) AS bonus  -- Returns NULL instead of 0
FROM employees;

-- Useful for avoiding division by zero:
SELECT
    sales / NULLIF(returns, 0) AS ratio  -- Returns NULL instead of error
FROM sales_data;

-- IFNULL / ISNULL (MySQL / SQL Server)
SELECT IFNULL(phone, 'N/A') FROM employees;      -- MySQL
SELECT ISNULL(phone, 'N/A') FROM employees;      -- SQL Server

-- NVL (Oracle)
SELECT NVL(phone, 'N/A') FROM employees;
```

---

## 9. String Functions in SELECT

```sql
-- Length
SELECT first_name, LENGTH(first_name) AS name_length FROM employees;

-- Case conversion
SELECT UPPER(first_name), LOWER(last_name) FROM employees;
SELECT INITCAP(first_name) FROM employees;  -- PostgreSQL, Oracle

-- Substring
SELECT SUBSTRING(phone FROM 1 FOR 3) AS area_code FROM employees;  -- PostgreSQL
SELECT SUBSTR(phone, 1, 3) AS area_code FROM employees;            -- Oracle, MySQL

-- Trim
SELECT TRIM(first_name) FROM employees;
SELECT LTRIM(first_name), RTRIM(last_name) FROM employees;
SELECT TRIM(BOTH ' ' FROM first_name) FROM employees;

-- Replace
SELECT REPLACE(phone, '-', '') AS phone_digits FROM employees;

-- Position/Locate
SELECT POSITION('@' IN email) AS at_position FROM employees;        -- PostgreSQL
SELECT LOCATE('@', email) AS at_position FROM employees;            -- MySQL

-- Padding
SELECT LPAD(employee_id::text, 5, '0') AS emp_code FROM employees;  -- '00123'
SELECT RPAD(first_name, 20, '.') FROM employees;                    -- 'John................'
```

---

## 10. Date/Time Functions

```sql
-- Current date/time
SELECT CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP;
SELECT NOW();  -- PostgreSQL, MySQL

-- Extract parts
SELECT
    hire_date,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month,
    EXTRACT(DAY FROM hire_date) AS hire_day
FROM employees;

-- Date arithmetic
SELECT
    hire_date,
    hire_date + INTERVAL '1 year' AS anniversary,    -- PostgreSQL
    DATE_ADD(hire_date, INTERVAL 1 YEAR) AS anniversary  -- MySQL
FROM employees;

-- Date difference
SELECT
    AGE(CURRENT_DATE, hire_date) AS tenure,           -- PostgreSQL
    DATEDIFF(CURRENT_DATE, hire_date) AS days_worked  -- MySQL
FROM employees;

-- Formatting
SELECT
    TO_CHAR(hire_date, 'Month DD, YYYY') AS formatted  -- PostgreSQL, Oracle
FROM employees;

SELECT
    DATE_FORMAT(hire_date, '%M %d, %Y') AS formatted   -- MySQL
FROM employees;
```

---

## 11. Numeric Functions

```sql
-- Rounding
SELECT
    ROUND(salary / 12, 2) AS monthly_salary,
    FLOOR(salary / 12) AS monthly_floor,
    CEIL(salary / 12) AS monthly_ceil,
    TRUNC(salary / 12, 2) AS monthly_truncated  -- PostgreSQL, Oracle
FROM employees;

-- Absolute value
SELECT ABS(balance) FROM accounts;

-- Mathematical functions
SELECT
    POWER(2, 10) AS power_result,        -- 1024
    SQRT(144) AS square_root,            -- 12
    MOD(17, 5) AS modulo,                -- 2 (remainder)
    GREATEST(10, 20, 30) AS max_value,   -- 30
    LEAST(10, 20, 30) AS min_value       -- 10
FROM dual;  -- Oracle syntax; omit FROM in PostgreSQL

-- Random
SELECT RANDOM();           -- PostgreSQL (0 to 1)
SELECT RAND();             -- MySQL (0 to 1)
SELECT FLOOR(RANDOM() * 100);  -- Random integer 0-99
```

---

## 12. Type Casting

```sql
-- PostgreSQL
SELECT
    '123'::INTEGER AS int_val,
    123::TEXT AS text_val,
    '2024-01-15'::DATE AS date_val,
    salary::NUMERIC(10,2) AS formatted_salary
FROM employees;

-- Standard SQL CAST
SELECT
    CAST('123' AS INTEGER) AS int_val,
    CAST(123 AS VARCHAR(10)) AS text_val,
    CAST('2024-01-15' AS DATE) AS date_val
FROM employees;

-- MySQL CONVERT
SELECT CONVERT('123', SIGNED INTEGER);
SELECT CONVERT(hire_date, CHAR);
```

---

## 13. Summary

| Clause | Purpose | Example |
|--------|---------|---------|
| SELECT | Choose columns | `SELECT first_name, salary` |
| FROM | Specify table | `FROM employees` |
| WHERE | Filter rows | `WHERE salary > 50000` |
| ORDER BY | Sort results | `ORDER BY salary DESC` |
| LIMIT | Restrict rows | `LIMIT 10 OFFSET 20` |
| DISTINCT | Remove duplicates | `SELECT DISTINCT dept_id` |
| CASE | Conditional logic | `CASE WHEN ... THEN ... END` |

**Key Points:**
- Understand execution order (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT)
- Use appropriate operators (BETWEEN, IN, LIKE, IS NULL)
- Learn your database's specific functions and syntax variations
- Use CASE for conditional logic in queries
- Handle NULLs properly with COALESCE or NULLIF
