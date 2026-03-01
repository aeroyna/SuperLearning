# Subqueries and CTEs

## 1. Introduction

**Subqueries** are queries nested inside other queries. **CTEs (Common Table Expressions)** provide a named, reusable way to define temporary result sets.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SUBQUERIES vs CTEs                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SUBQUERIES:                                                               │
│   • Nested inside another query                                             │
│   • Can appear in SELECT, FROM, WHERE, HAVING                               │
│   • Executed for each row (correlated) or once (non-correlated)            │
│   • Can be harder to read when deeply nested                                │
│                                                                              │
│   CTEs (Common Table Expressions):                                          │
│   • Defined with WITH clause before main query                              │
│   • Named and reusable within the query                                     │
│   • Better readability for complex queries                                  │
│   • Support recursion                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# Part 1: Subqueries

## 2. Scalar Subqueries

Scalar subqueries return a single value (one row, one column).

### 2.1 In SELECT Clause

```sql
-- Get employee with their department's average salary
SELECT
    first_name,
    last_name,
    salary,
    department_id,
    (SELECT AVG(salary) FROM employees e2
     WHERE e2.department_id = e1.department_id) AS dept_avg_salary
FROM employees e1;

-- Compare to company average
SELECT
    first_name,
    salary,
    salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;
```

### 2.2 In WHERE Clause

```sql
-- Employees earning more than average
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Employee with highest salary
SELECT * FROM employees
WHERE salary = (SELECT MAX(salary) FROM employees);

-- Most recent order
SELECT * FROM orders
WHERE order_date = (SELECT MAX(order_date) FROM orders);
```

---

## 3. Multi-Row Subqueries

Subqueries returning multiple values (one column, multiple rows).

### 3.1 Using IN

```sql
-- Employees in departments located in 'New York'
SELECT * FROM employees
WHERE department_id IN (
    SELECT department_id FROM departments
    WHERE location = 'New York'
);

-- Products that have been ordered
SELECT * FROM products
WHERE product_id IN (
    SELECT DISTINCT product_id FROM order_items
);

-- Products never ordered
SELECT * FROM products
WHERE product_id NOT IN (
    SELECT product_id FROM order_items
    WHERE product_id IS NOT NULL  -- Important! NULL handling
);
```

### 3.2 Using ANY/SOME

```sql
-- Employees earning more than ANY employee in department 10
SELECT * FROM employees
WHERE salary > ANY (
    SELECT salary FROM employees WHERE department_id = 10
);
-- Same as: salary > MIN(salary in dept 10)

-- Products cheaper than any product in 'Electronics'
SELECT * FROM products
WHERE price < ANY (
    SELECT price FROM products WHERE category = 'Electronics'
);
```

### 3.3 Using ALL

```sql
-- Employees earning more than ALL employees in department 10
SELECT * FROM employees
WHERE salary > ALL (
    SELECT salary FROM employees WHERE department_id = 10
);
-- Same as: salary > MAX(salary in dept 10)

-- Highest salary in each department
SELECT * FROM employees e1
WHERE salary >= ALL (
    SELECT salary FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

---

## 4. Multi-Column Subqueries

Subqueries returning multiple columns.

```sql
-- Employees with the same job and department as employee 100
SELECT * FROM employees
WHERE (job_id, department_id) IN (
    SELECT job_id, department_id FROM employees WHERE employee_id = 100
);

-- Latest order for each customer
SELECT * FROM orders
WHERE (customer_id, order_date) IN (
    SELECT customer_id, MAX(order_date)
    FROM orders
    GROUP BY customer_id
);
```

---

## 5. Correlated Subqueries

Correlated subqueries reference the outer query and are executed for each row.

```sql
-- Employees earning above their department's average
SELECT first_name, last_name, salary, department_id
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id  -- Reference to outer query
);

-- Products with price above category average
SELECT product_name, price, category
FROM products p1
WHERE price > (
    SELECT AVG(price)
    FROM products p2
    WHERE p2.category = p1.category
);

-- Orders with above-average items count
SELECT order_id, customer_id
FROM orders o
WHERE (SELECT COUNT(*) FROM order_items oi WHERE oi.order_id = o.order_id)
    > (SELECT AVG(item_count) FROM (
        SELECT COUNT(*) AS item_count FROM order_items GROUP BY order_id
    ) counts);
```

---

## 6. EXISTS Operator

EXISTS tests whether a subquery returns any rows.

```sql
-- Customers who have placed orders
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Customers who have never ordered
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Departments with at least one high earner
SELECT * FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.department_id = d.department_id AND e.salary > 100000
);
```

### EXISTS vs IN

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       EXISTS vs IN                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   EXISTS:                                                                   │
│   • Stops as soon as it finds ONE match                                    │
│   • Better for large subquery results                                      │
│   • Handles NULL values correctly                                          │
│   • Must be correlated                                                     │
│                                                                              │
│   IN:                                                                       │
│   • Evaluates entire subquery first                                        │
│   • Better for small subquery results                                      │
│   • Beware of NULL values (NOT IN with NULLs returns no rows)             │
│   • Can be non-correlated                                                  │
│                                                                              │
│   NULL handling difference:                                                 │
│   SELECT * FROM t WHERE x NOT IN (1, 2, NULL);  -- Returns NO rows!        │
│   SELECT * FROM t WHERE NOT EXISTS (...);       -- Works correctly         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Subqueries in FROM Clause (Derived Tables)

```sql
-- Treat subquery result as a table
SELECT dept_name, avg_salary
FROM (
    SELECT
        d.department_name AS dept_name,
        AVG(e.salary) AS avg_salary
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    GROUP BY d.department_name
) AS dept_averages
WHERE avg_salary > 50000;

-- Pagination with row numbers (older method)
SELECT * FROM (
    SELECT
        e.*,
        ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
    FROM employees e
) numbered
WHERE rn BETWEEN 11 AND 20;

-- Complex aggregation
SELECT
    category,
    total_sales,
    total_sales / grand_total * 100 AS percentage
FROM (
    SELECT category, SUM(amount) AS total_sales
    FROM sales
    GROUP BY category
) category_sales
CROSS JOIN (
    SELECT SUM(amount) AS grand_total FROM sales
) totals;
```

---

## 8. Subqueries in Other Clauses

### 8.1 In INSERT

```sql
-- Insert from subquery
INSERT INTO high_earners (employee_id, name, salary)
SELECT employee_id, first_name || ' ' || last_name, salary
FROM employees
WHERE salary > 100000;

-- Copy table structure and data
INSERT INTO employees_backup
SELECT * FROM employees;
```

### 8.2 In UPDATE

```sql
-- Update with subquery value
UPDATE employees
SET salary = (SELECT AVG(salary) FROM employees)
WHERE employee_id = 100;

-- Update based on subquery condition
UPDATE products
SET discount = 0.1
WHERE product_id IN (
    SELECT product_id FROM order_items
    GROUP BY product_id
    HAVING COUNT(*) > 100
);

-- Update with correlated subquery (PostgreSQL)
UPDATE employees e
SET salary = (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
)
WHERE department_id = 10;
```

### 8.3 In DELETE

```sql
-- Delete based on subquery
DELETE FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders
);

-- Delete with EXISTS
DELETE FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
);
```

---

# Part 2: Common Table Expressions (CTEs)

## 9. Basic CTE Syntax

```sql
-- Basic CTE
WITH department_stats AS (
    SELECT
        department_id,
        COUNT(*) AS emp_count,
        AVG(salary) AS avg_salary,
        SUM(salary) AS total_salary
    FROM employees
    GROUP BY department_id
)
SELECT
    d.department_name,
    ds.emp_count,
    ds.avg_salary
FROM department_stats ds
JOIN departments d ON ds.department_id = d.department_id
WHERE ds.emp_count > 5;
```

---

## 10. Multiple CTEs

```sql
-- Multiple CTEs separated by commas
WITH
    high_earners AS (
        SELECT * FROM employees WHERE salary > 100000
    ),
    department_counts AS (
        SELECT department_id, COUNT(*) AS emp_count
        FROM employees
        GROUP BY department_id
    ),
    large_departments AS (
        SELECT department_id FROM department_counts WHERE emp_count > 10
    )
SELECT
    he.first_name,
    he.last_name,
    he.salary,
    d.department_name
FROM high_earners he
JOIN departments d ON he.department_id = d.department_id
WHERE he.department_id IN (SELECT department_id FROM large_departments);
```

---

## 11. CTEs Referencing Other CTEs

```sql
-- CTEs can reference earlier CTEs
WITH
    monthly_sales AS (
        SELECT
            DATE_TRUNC('month', order_date) AS month,
            SUM(total) AS total_sales
        FROM orders
        GROUP BY DATE_TRUNC('month', order_date)
    ),
    sales_with_growth AS (
        SELECT
            month,
            total_sales,
            LAG(total_sales) OVER (ORDER BY month) AS prev_month_sales
        FROM monthly_sales
    ),
    growth_rates AS (
        SELECT
            month,
            total_sales,
            prev_month_sales,
            CASE
                WHEN prev_month_sales IS NOT NULL AND prev_month_sales > 0
                THEN (total_sales - prev_month_sales) / prev_month_sales * 100
                ELSE NULL
            END AS growth_rate
        FROM sales_with_growth
    )
SELECT * FROM growth_rates
ORDER BY month;
```

---

## 12. Recursive CTEs

Recursive CTEs are powerful for hierarchical or iterative data.

### 12.1 Basic Recursive Structure

```sql
-- Recursive CTE structure
WITH RECURSIVE cte_name AS (
    -- Anchor member (base case)
    SELECT ... FROM table WHERE condition

    UNION ALL  -- or UNION

    -- Recursive member (references cte_name)
    SELECT ... FROM table
    JOIN cte_name ON ...
)
SELECT * FROM cte_name;
```

### 12.2 Organizational Hierarchy

```sql
-- Employee hierarchy (manager-subordinate)
WITH RECURSIVE org_hierarchy AS (
    -- Anchor: Start with top-level manager (no manager)
    SELECT
        employee_id,
        first_name,
        manager_id,
        1 AS level,
        first_name::text AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: Find employees reporting to current level
    SELECT
        e.employee_id,
        e.first_name,
        e.manager_id,
        oh.level + 1,
        oh.path || ' > ' || e.first_name
    FROM employees e
    JOIN org_hierarchy oh ON e.manager_id = oh.employee_id
)
SELECT * FROM org_hierarchy
ORDER BY path;
```

### 12.3 Category Tree

```sql
-- Categories with subcategories
WITH RECURSIVE category_tree AS (
    -- Root categories
    SELECT
        category_id,
        name,
        parent_id,
        name::text AS full_path,
        0 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Child categories
    SELECT
        c.category_id,
        c.name,
        c.parent_id,
        ct.full_path || ' / ' || c.name,
        ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT * FROM category_tree
ORDER BY full_path;
```

### 12.4 Number Series

```sql
-- Generate numbers 1 to 100
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers;

-- Generate date series
WITH RECURSIVE dates AS (
    SELECT DATE '2024-01-01' AS date
    UNION ALL
    SELECT date + INTERVAL '1 day' FROM dates WHERE date < '2024-12-31'
)
SELECT * FROM dates;
```

### 12.5 Graph Traversal

```sql
-- Find all paths between nodes
WITH RECURSIVE paths AS (
    -- Start from source node
    SELECT
        source AS start_node,
        target AS end_node,
        ARRAY[source, target] AS path,
        1 AS hops
    FROM edges
    WHERE source = 1  -- Starting node

    UNION ALL

    -- Extend paths
    SELECT
        p.start_node,
        e.target,
        p.path || e.target,
        p.hops + 1
    FROM paths p
    JOIN edges e ON p.end_node = e.source
    WHERE NOT e.target = ANY(p.path)  -- Prevent cycles
      AND p.hops < 10  -- Limit depth
)
SELECT * FROM paths
WHERE end_node = 5;  -- Target node
```

---

## 13. CTE with INSERT, UPDATE, DELETE

### 13.1 CTE with INSERT

```sql
-- Insert using CTE results
WITH top_products AS (
    SELECT product_id, product_name, total_sales
    FROM (
        SELECT
            p.product_id,
            p.product_name,
            SUM(oi.quantity * oi.price) AS total_sales
        FROM products p
        JOIN order_items oi ON p.product_id = oi.product_id
        GROUP BY p.product_id, p.product_name
    ) sales
    WHERE total_sales > 10000
)
INSERT INTO featured_products (product_id, product_name, reason)
SELECT product_id, product_name, 'High sales volume'
FROM top_products;
```

### 13.2 CTE with UPDATE

```sql
-- Update using CTE
WITH avg_salaries AS (
    SELECT department_id, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
)
UPDATE employees e
SET salary = salary * 1.1
FROM avg_salaries a
WHERE e.department_id = a.department_id
  AND e.salary < a.avg_sal * 0.8;  -- Underpaid employees
```

### 13.3 CTE with DELETE (PostgreSQL)

```sql
-- Delete using CTE with RETURNING
WITH deleted AS (
    DELETE FROM orders
    WHERE order_date < '2020-01-01'
    RETURNING *
)
INSERT INTO archived_orders
SELECT * FROM deleted;
```

---

## 14. Subquery vs CTE Comparison

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SUBQUERY vs CTE COMPARISON                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SUBQUERY:                                                                 │
│   ✓ Works in all SQL dialects                                              │
│   ✓ Can be more efficient for simple cases                                 │
│   ✗ Can't be reused in same query                                          │
│   ✗ Nested subqueries hard to read                                         │
│   ✗ No recursion support                                                   │
│                                                                              │
│   CTE:                                                                      │
│   ✓ Named and self-documenting                                             │
│   ✓ Reusable within query                                                  │
│   ✓ Supports recursion                                                     │
│   ✓ Better readability                                                     │
│   ✗ Not all databases support CTEs (older versions)                        │
│   ✗ May be less optimized in some databases                               │
│                                                                              │
│   WHEN TO USE EACH:                                                         │
│   • Simple, one-time use → Subquery                                        │
│   • Complex, multi-step logic → CTE                                        │
│   • Need recursion → CTE                                                   │
│   • Same result used multiple times → CTE                                  │
│   • Older database → Subquery                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 15. Performance Considerations

```sql
-- CTEs may be materialized (computed once) or inlined
-- This varies by database

-- PostgreSQL 12+ allows control
WITH cte AS MATERIALIZED (
    SELECT * FROM large_table WHERE condition
)
SELECT * FROM cte t1 JOIN cte t2 ON t1.id = t2.parent_id;

WITH cte AS NOT MATERIALIZED (
    SELECT * FROM small_table
)
SELECT * FROM cte WHERE condition;

-- For performance-critical queries, check execution plans:
EXPLAIN ANALYZE
WITH complex_cte AS (...)
SELECT * FROM complex_cte;
```

---

## 16. Summary

| Concept | Description | Example |
|---------|-------------|---------|
| Scalar Subquery | Returns single value | `WHERE x = (SELECT MAX(y) FROM t)` |
| Multi-row Subquery | Returns multiple values | `WHERE x IN (SELECT y FROM t)` |
| Correlated Subquery | References outer query | `WHERE x > (SELECT AVG(y) FROM t2 WHERE t2.id = t1.id)` |
| EXISTS | Tests for existence | `WHERE EXISTS (SELECT 1 FROM t WHERE ...)` |
| Derived Table | Subquery in FROM | `FROM (SELECT ...) AS alias` |
| Basic CTE | Named temporary result | `WITH name AS (...) SELECT ...` |
| Recursive CTE | Self-referencing CTE | `WITH RECURSIVE ... UNION ALL ...` |

**Key Points:**
- Use subqueries for simple, one-time lookups
- Use CTEs for complex, multi-step queries
- Recursive CTEs are essential for hierarchical data
- Watch out for NULL handling with NOT IN
- EXISTS is often more efficient than IN for large datasets
- Check execution plans for performance-critical queries
