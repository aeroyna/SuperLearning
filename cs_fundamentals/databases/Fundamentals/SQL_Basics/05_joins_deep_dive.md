# Joins Deep Dive

## 1. Introduction

Joins combine rows from two or more tables based on related columns. Understanding joins is fundamental to working with relational databases.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           JOIN TYPES OVERVIEW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   INNER JOIN      - Only matching rows from both tables                     │
│   LEFT JOIN       - All rows from left + matching from right                │
│   RIGHT JOIN      - All rows from right + matching from left                │
│   FULL OUTER JOIN - All rows from both tables                               │
│   CROSS JOIN      - Cartesian product (all combinations)                    │
│   SELF JOIN       - Table joined with itself                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Sample Data

```sql
-- Employees table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    manager_id INT
);

INSERT INTO employees VALUES
    (1, 'Alice', 1, NULL),
    (2, 'Bob', 1, 1),
    (3, 'Charlie', 2, 1),
    (4, 'Diana', NULL, 2),
    (5, 'Eve', 3, 3);

-- Departments table
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    budget DECIMAL(10, 2)
);

INSERT INTO departments VALUES
    (1, 'Engineering', 500000),
    (2, 'Marketing', 300000),
    (4, 'Finance', 400000);

-- Note: Employee Diana has no department (NULL)
-- Note: Department Finance (id=4) has no employees
-- Note: Employee Eve's department (id=3) doesn't exist
```

```
┌──────────────────────────────┐     ┌──────────────────────────────┐
│         employees            │     │        departments           │
├────┬─────────┬────────┬──────┤     ├────┬─────────────┬───────────┤
│ id │  name   │dept_id │mgr_id│     │ id │    name     │  budget   │
├────┼─────────┼────────┼──────┤     ├────┼─────────────┼───────────┤
│ 1  │ Alice   │   1    │ NULL │     │ 1  │ Engineering │  500000   │
│ 2  │ Bob     │   1    │  1   │     │ 2  │ Marketing   │  300000   │
│ 3  │ Charlie │   2    │  1   │     │ 4  │ Finance     │  400000   │
│ 4  │ Diana   │  NULL  │  2   │     └────┴─────────────┴───────────┘
│ 5  │ Eve     │   3    │  3   │
└────┴─────────┴────────┴──────┘
```

---

## 3. INNER JOIN

Returns only rows with matches in both tables.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            INNER JOIN                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Table A          Table B          Result                                   │
│   ┌───┐           ┌───┐            ┌───┬───┐                                │
│   │ 1 │           │ 1 │            │ 1 │ 1 │                                │
│   │ 2 │    ⋈      │ 2 │     =      │ 2 │ 2 │                                │
│   │ 3 │           │ 4 │            └───┴───┘                                │
│   └───┘           └───┘                                                      │
│                                                                              │
│   Only matching values (1, 2) are included                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- Basic INNER JOIN
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- Result:
-- | name    | department  |
-- |---------|-------------|
-- | Alice   | Engineering |
-- | Bob     | Engineering |
-- | Charlie | Marketing   |

-- Note: Diana (no dept) and Eve (dept doesn't exist) excluded
-- Note: Finance dept (no employees) excluded

-- Equivalent using WHERE (old syntax - not recommended)
SELECT e.name, d.name AS department
FROM employees e, departments d
WHERE e.department_id = d.id;

-- Multiple conditions
SELECT e.name, d.name AS department
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.id
    AND d.budget > 200000;
```

---

## 4. LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the left table, with matching rows from the right (or NULL).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            LEFT JOIN                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Table A          Table B          Result                                   │
│   ┌───┐           ┌───┐            ┌───┬──────┐                             │
│   │ 1 │           │ 1 │            │ 1 │  1   │                             │
│   │ 2 │    ⟕      │ 2 │     =      │ 2 │  2   │                             │
│   │ 3 │           │ 4 │            │ 3 │ NULL │                             │
│   └───┘           └───┘            └───┴──────┘                             │
│                                                                              │
│   All rows from A, matching from B (NULL if no match)                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- LEFT JOIN
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- Result:
-- | name    | department  |
-- |---------|-------------|
-- | Alice   | Engineering |
-- | Bob     | Engineering |
-- | Charlie | Marketing   |
-- | Diana   | NULL        |  ← No department
-- | Eve     | NULL        |  ← Dept doesn't exist

-- Find employees without a valid department
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;

-- Result:
-- | name  |
-- |-------|
-- | Diana |
-- | Eve   |
```

---

## 5. RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from the right table, with matching rows from the left (or NULL).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            RIGHT JOIN                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Table A          Table B          Result                                   │
│   ┌───┐           ┌───┐            ┌──────┬───┐                             │
│   │ 1 │           │ 1 │            │  1   │ 1 │                             │
│   │ 2 │    ⟖      │ 2 │     =      │  2   │ 2 │                             │
│   │ 3 │           │ 4 │            │ NULL │ 4 │                             │
│   └───┘           └───┘            └──────┴───┘                             │
│                                                                              │
│   All rows from B, matching from A (NULL if no match)                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- RIGHT JOIN
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;

-- Result:
-- | name    | department  |
-- |---------|-------------|
-- | Alice   | Engineering |
-- | Bob     | Engineering |
-- | Charlie | Marketing   |
-- | NULL    | Finance     |  ← No employees in Finance

-- Find departments with no employees
SELECT d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL;

-- Result:
-- | department |
-- |------------|
-- | Finance    |

-- Note: RIGHT JOIN can always be rewritten as LEFT JOIN
SELECT d.name AS department
FROM departments d
LEFT JOIN employees e ON e.department_id = d.id
WHERE e.id IS NULL;
```

---

## 6. FULL OUTER JOIN

Returns all rows from both tables, with NULL where there's no match.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          FULL OUTER JOIN                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Table A          Table B          Result                                   │
│   ┌───┐           ┌───┐            ┌──────┬──────┐                          │
│   │ 1 │           │ 1 │            │  1   │  1   │                          │
│   │ 2 │    ⟗      │ 2 │     =      │  2   │  2   │                          │
│   │ 3 │           │ 4 │            │  3   │ NULL │                          │
│   └───┘           └───┘            │ NULL │  4   │                          │
│                                    └──────┴──────┘                          │
│                                                                              │
│   All rows from both A and B                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```sql
-- FULL OUTER JOIN
SELECT e.name, d.name AS department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- Result:
-- | name    | department  |
-- |---------|-------------|
-- | Alice   | Engineering |
-- | Bob     | Engineering |
-- | Charlie | Marketing   |
-- | Diana   | NULL        |  ← Employee without valid dept
-- | Eve     | NULL        |  ← Employee without valid dept
-- | NULL    | Finance     |  ← Dept without employees

-- Find unmatched on either side
SELECT
    COALESCE(e.name, 'NO EMPLOYEE') AS employee,
    COALESCE(d.name, 'NO DEPARTMENT') AS department
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL OR d.id IS NULL;

-- MySQL doesn't support FULL OUTER JOIN directly
-- Use UNION of LEFT and RIGHT JOIN:
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
UNION
SELECT e.name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

---

## 7. CROSS JOIN

Returns the Cartesian product - every row from A paired with every row from B.

```sql
-- CROSS JOIN
SELECT e.name, d.name AS department
FROM employees e
CROSS JOIN departments d;

-- Result: 5 employees × 3 departments = 15 rows
-- | name    | department  |
-- |---------|-------------|
-- | Alice   | Engineering |
-- | Alice   | Marketing   |
-- | Alice   | Finance     |
-- | Bob     | Engineering |
-- | Bob     | Marketing   |
-- | Bob     | Finance     |
-- ... (15 rows total)

-- Use case: Generate all combinations
SELECT
    sizes.size,
    colors.color,
    sizes.size || '-' || colors.color AS sku
FROM (VALUES ('S'), ('M'), ('L'), ('XL')) AS sizes(size)
CROSS JOIN (VALUES ('Red'), ('Blue'), ('Green')) AS colors(color);

-- Result: 4 sizes × 3 colors = 12 SKUs
```

---

## 8. SELF JOIN

Joins a table with itself - useful for hierarchical data.

```sql
-- Find employees and their managers
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;

-- Result:
-- | employee | manager |
-- |----------|---------|
-- | Alice    | NULL    |
-- | Bob      | Alice   |
-- | Charlie  | Alice   |
-- | Diana    | Bob     |
-- | Eve      | Charlie |

-- Find employees who manage others
SELECT DISTINCT m.name AS manager
FROM employees e
INNER JOIN employees m ON e.manager_id = m.id;

-- Result:
-- | manager |
-- |---------|
-- | Alice   |
-- | Bob     |
-- | Charlie |

-- Full hierarchy (recursive CTE)
WITH RECURSIVE org_chart AS (
    -- Base case: top-level (no manager)
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT
    REPEAT('  ', level) || name AS org_chart,
    level
FROM org_chart
ORDER BY level, name;
```

---

## 9. Multiple Joins

```sql
-- Orders with customer and product details
SELECT
    o.id AS order_id,
    c.name AS customer,
    p.name AS product,
    oi.quantity,
    oi.price
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
WHERE o.created_at > '2024-01-01';

-- Mix of join types
SELECT
    c.name AS customer,
    COUNT(o.id) AS total_orders,
    COALESCE(SUM(o.total), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
LEFT JOIN payments p ON o.id = p.order_id AND p.status = 'completed'
GROUP BY c.id, c.name;

-- Complex join with conditions
SELECT
    e.name AS employee,
    d.name AS department,
    m.name AS manager,
    p.name AS project
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
LEFT JOIN employees m ON e.manager_id = m.id
LEFT JOIN project_assignments pa ON e.id = pa.employee_id
    AND pa.end_date IS NULL  -- Only active assignments
LEFT JOIN projects p ON pa.project_id = p.id;
```

---

## 10. Join Conditions and Filtering

```sql
-- Join condition vs WHERE clause
-- Join condition: Affects which rows are joined
-- WHERE clause: Filters final result

-- These are DIFFERENT for LEFT JOIN:

-- 1. Condition in JOIN (keeps all left rows)
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d
    ON e.department_id = d.id
    AND d.budget > 400000;
-- Returns ALL employees, department only if budget > 400000

-- 2. Condition in WHERE (filters after join)
SELECT e.name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.budget > 400000;
-- Returns only employees with matching dept with budget > 400000

-- For INNER JOIN, they're equivalent
-- But for OUTER joins, placement matters!

-- Multiple join conditions
SELECT *
FROM orders o
JOIN order_items oi
    ON o.id = oi.order_id
    AND oi.quantity > 0
    AND oi.price > 0;

-- Non-equi join (range join)
SELECT
    e.name,
    e.salary,
    sg.grade
FROM employees e
JOIN salary_grades sg
    ON e.salary BETWEEN sg.min_salary AND sg.max_salary;
```

---

## 11. Performance Considerations

```sql
-- Index usage
-- Ensure join columns are indexed
CREATE INDEX idx_employees_department ON employees(department_id);
CREATE INDEX idx_orders_customer ON orders(customer_id);

-- Avoid functions on join columns
-- BAD: Prevents index usage
SELECT * FROM users u
JOIN orders o ON LOWER(u.email) = LOWER(o.customer_email);

-- GOOD: Compare directly
SELECT * FROM users u
JOIN orders o ON u.email = o.customer_email;

-- Reduce rows before joining
-- BAD: Join all, then filter
SELECT *
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
WHERE o.created_at > '2024-01-01';

-- BETTER: Filter first (CTE or subquery)
WITH recent_orders AS (
    SELECT * FROM orders WHERE created_at > '2024-01-01'
)
SELECT *
FROM recent_orders o
JOIN order_items oi ON o.id = oi.order_id;

-- EXPLAIN to understand join strategy
EXPLAIN ANALYZE
SELECT e.name, d.name
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

---

## 12. Join Visual Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         JOIN TYPES VISUAL                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   INNER JOIN           LEFT JOIN            RIGHT JOIN                      │
│      ┌───┐               ┌───┐                ┌───┐                         │
│     ╱     ╲             ╱█████╲              ╱█████╲                        │
│    ╱   ███╲ ╲          ╱███████╲            ╱  ███╲ ╲                       │
│   │   █████│ │        │█████████│          │  █████│ │                      │
│    ╲   ███╱ ╱          ╲███████╱            ╲  ███╱ ╱                       │
│     ╲     ╱             ╲█████╱              ╲█████╱                        │
│      └───┘               └───┘                └───┘                         │
│    Only overlap        All left +          All right +                      │
│                        matching right      matching left                    │
│                                                                              │
│   FULL OUTER JOIN      CROSS JOIN          SELF JOIN                        │
│      ┌───┐               ┌───┐              ┌───┐                           │
│     ╱█████╲             │ × │              │ ⟲ │                           │
│    ╱███████╲            │   │              │   │                            │
│   │█████████│           └───┘              └───┘                            │
│    ╲███████╱          Every row from      Table joined                      │
│     ╲█████╱           A with every        with itself                       │
│      └───┘            row from B                                            │
│    All from both                                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 13. Summary

| Join Type | Returns | Use Case |
|-----------|---------|----------|
| INNER | Only matching rows | Default for related data |
| LEFT | All left + matching right | Include records with missing related data |
| RIGHT | All right + matching left | Less common, prefer LEFT |
| FULL OUTER | All from both | Find all unmatched on both sides |
| CROSS | All combinations | Generate combinations |
| SELF | Table with itself | Hierarchies, comparisons |

**Key Points:**
- Always specify join type explicitly (don't rely on defaults)
- Index columns used in join conditions
- Consider join order for complex queries
- Use EXPLAIN to analyze join performance
- Remember: WHERE vs ON placement matters for OUTER joins
