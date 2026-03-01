# Third Normal Form (3NF)

## 1. Introduction

**Third Normal Form (3NF)** builds on 2NF by eliminating **transitive dependencies**. It ensures that non-key attributes depend directly on the primary key, not through other non-key attributes.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        3NF REQUIREMENTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A table is in 3NF if:                                                     │
│                                                                              │
│   1. It is in 2NF                                                           │
│   2. No non-key attribute is TRANSITIVELY dependent on the primary key     │
│                                                                              │
│   Alternative definition (more precise):                                    │
│   For every non-trivial functional dependency X → A:                       │
│   • X is a superkey, OR                                                     │
│   • A is part of some candidate key (prime attribute)                       │
│                                                                              │
│   Simply: "Every non-key attribute depends on the key,                      │
│            the whole key, and nothing but the key"                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Understanding Transitive Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     TRANSITIVE DEPENDENCY                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   If A → B and B → C, then A → C transitively                               │
│                                                                              │
│   Example:                                                                   │
│   emp_id → dept_id         (employee determines department)                │
│   dept_id → dept_name      (department determines department name)         │
│   ─────────────────────────────────────────────                             │
│   emp_id → dept_name       (transitive - through dept_id)                   │
│                                                                              │
│   The problem: dept_name depends on dept_id, not directly on emp_id        │
│                                                                              │
│   ┌────────┐      ┌─────────┐      ┌───────────┐                           │
│   │ emp_id │─────▶│ dept_id │─────▶│ dept_name │                           │
│   └────────┘      └─────────┘      └───────────┘                           │
│       PK            Non-key         Non-key                                 │
│                   attribute        attribute                                 │
│                                                                              │
│   dept_name is transitively dependent on emp_id (via dept_id)              │
│   This violates 3NF                                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 3NF Violation Example

### 3.1 Problematic Table

```sql
-- Employees table (violates 3NF)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    dept_name VARCHAR(100),    -- Transitively dependent via dept_id
    dept_location VARCHAR(100), -- Transitively dependent via dept_id
    dept_budget DECIMAL(12, 2) -- Transitively dependent via dept_id
);
```

```
┌────────┬──────────┬─────────┬─────────────┬───────────────┬────────────┐
│ emp_id │ emp_name │ dept_id │  dept_name  │ dept_location │ dept_budget│
├────────┼──────────┼─────────┼─────────────┼───────────────┼────────────┤
│  101   │  Alice   │   10    │ Engineering │   Building A  │  500000    │
│  102   │   Bob    │   10    │ Engineering │   Building A  │  500000    │
│  103   │  Carol   │   10    │ Engineering │   Building A  │  500000    │
│  104   │   Dave   │   20    │    Sales    │   Building B  │  300000    │
│  105   │   Eve    │   20    │    Sales    │   Building B  │  300000    │
│  106   │  Frank   │   30    │     HR      │   Building C  │  200000    │
└────────┴──────────┴─────────┴─────────────┴───────────────┴────────────┘

Functional Dependencies:
• emp_id → emp_name, dept_id
• dept_id → dept_name, dept_location, dept_budget
• emp_id → dept_name, dept_location, dept_budget (transitively)

Problems:

REDUNDANCY:
• "Engineering", "Building A", "500000" stored 3 times
• Every department detail repeated for each employee

UPDATE ANOMALY:
• To move Engineering to Building D, update 3 rows
• Risk of inconsistency if some rows missed

INSERT ANOMALY:
• Can't add a new department until we hire an employee

DELETE ANOMALY:
• If Frank leaves, we lose all information about HR department
```

---

## 4. Converting to 3NF

### 4.1 Identify Dependencies

```
Primary Key: emp_id

Direct Dependencies (okay):
• emp_id → emp_name
• emp_id → dept_id

Transitive Dependencies (violate 3NF):
• dept_id → dept_name, dept_location, dept_budget
• Therefore: emp_id → dept_name (transitively via dept_id)
```

### 4.2 Decompose the Table

```sql
-- Step 1: Create Departments table (holds dept_id dependencies)
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL,
    dept_location VARCHAR(100),
    dept_budget DECIMAL(12, 2)
);

-- Step 2: Keep only direct dependencies in Employees
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL,
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

-- Insert data
INSERT INTO departments VALUES
    (10, 'Engineering', 'Building A', 500000),
    (20, 'Sales', 'Building B', 300000),
    (30, 'HR', 'Building C', 200000);

INSERT INTO employees VALUES
    (101, 'Alice', 10),
    (102, 'Bob', 10),
    (103, 'Carol', 10),
    (104, 'Dave', 20),
    (105, 'Eve', 20),
    (106, 'Frank', 30);
```

### 4.3 Result

```
┌───────────────┐              ┌──────────────────┐
│   employees   │              │   departments    │
├───────────────┤              ├──────────────────┤
│ emp_id (PK)   │              │ dept_id (PK)     │
│ emp_name      │              │ dept_name        │
│ dept_id (FK)  │─────────────►│ dept_location    │
└───────────────┘              │ dept_budget      │
                               └──────────────────┘

• Department info stored ONCE
• No redundancy
• Updates in one place
• Can add/delete departments independently
```

---

## 5. More Complex Example

### 5.1 Order with Shipping Information

```sql
-- Orders table (violates 3NF)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    shipping_zip VARCHAR(10),
    shipping_city VARCHAR(100),    -- Transitively dependent via zip
    shipping_state CHAR(2),        -- Transitively dependent via zip
    shipping_country VARCHAR(50)   -- Transitively dependent via zip
);
```

```
Dependencies:
• order_id → customer_id, order_date, shipping_zip
• shipping_zip → shipping_city, shipping_state, shipping_country
• order_id → shipping_city (transitively via shipping_zip)

Problem: Zip code info repeated for every order to that zip
```

### 5.2 Normalized Design

```sql
-- Zip codes table (lookup table)
CREATE TABLE zip_codes (
    zip VARCHAR(10) PRIMARY KEY,
    city VARCHAR(100) NOT NULL,
    state CHAR(2) NOT NULL,
    country VARCHAR(50) DEFAULT 'USA'
);

-- Orders table (now in 3NF)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    shipping_zip VARCHAR(10),
    FOREIGN KEY (shipping_zip) REFERENCES zip_codes(zip)
);

-- Query to get full address
SELECT
    o.order_id,
    o.order_date,
    z.city,
    z.state,
    o.shipping_zip
FROM orders o
JOIN zip_codes z ON o.shipping_zip = z.zip;
```

---

## 6. Practical Considerations

### 6.1 When NOT to Strictly Follow 3NF

```sql
-- Sometimes denormalization is acceptable for performance

-- Example: Storing customer name on orders
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    customer_name VARCHAR(100),  -- Denormalized!
    order_date DATE
);

-- Why might this be okay?
-- • Name rarely changes
-- • Historical accuracy (name at time of order)
-- • Avoids joins for common queries
-- • Read-heavy workload

-- Trade-off:
-- • Slight redundancy vs significant performance gain
-- • Must update both places if name changes
-- • Consider views or materialized views instead
```

### 6.2 The 3NF vs Performance Balance

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     3NF TRADE-OFFS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   FULL 3NF                         DENORMALIZED                             │
│   ─────────                        ────────────                              │
│   ✓ No redundancy                  ✓ Fewer joins                            │
│   ✓ Easy updates                   ✓ Faster reads                           │
│   ✓ Data integrity                 ✓ Simpler queries                        │
│   ✗ More joins                     ✗ Update complexity                      │
│   ✗ Complex queries                ✗ Potential inconsistency               │
│   ✗ Join overhead                  ✗ More storage                          │
│                                                                              │
│   RECOMMENDATIONS:                                                          │
│   • Start with 3NF                                                          │
│   • Measure performance                                                     │
│   • Denormalize selectively with evidence                                  │
│   • Document denormalization decisions                                      │
│   • Consider materialized views first                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Identifying 3NF Violations

### 7.1 Step-by-Step Process

```
1. Ensure table is in 2NF
2. Identify all non-key attributes
3. For each pair of non-key attributes A and B:
   - Does A determine B? (A → B)
   - If yes, this is a transitive dependency
4. Non-key → non-key = violation
```

### 7.2 SQL Detection

```sql
-- Check if dept_name varies independently of emp_id
-- but consistently with dept_id

-- First, check dept_id → dept_name holds
SELECT dept_id, COUNT(DISTINCT dept_name) as name_count
FROM employees
GROUP BY dept_id
HAVING COUNT(DISTINCT dept_name) > 1;
-- If returns rows: data inconsistency
-- If empty: dept_id → dept_name (transitive dependency exists)

-- Verify redundancy exists
SELECT dept_id, dept_name, COUNT(*) as row_count
FROM employees
GROUP BY dept_id, dept_name
HAVING COUNT(*) > 1;
-- Shows how many times each dept info is repeated
```

---

## 8. Common 3NF Violation Patterns

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  COMMON 3NF VIOLATION PATTERNS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. LOOKUP VALUES                                                           │
│      order_id → status_code → status_description                            │
│      Fix: Create status lookup table                                        │
│                                                                              │
│   2. DERIVED ADDRESS FIELDS                                                  │
│      address_id → zip_code → city, state, country                          │
│      Fix: Create geographic lookup table                                    │
│                                                                              │
│   3. CATEGORY HIERARCHIES                                                    │
│      product_id → subcategory → category → department                      │
│      Fix: Create category hierarchy tables                                  │
│                                                                              │
│   4. EMPLOYEE-DEPARTMENT-MANAGER                                            │
│      emp_id → dept_id → dept_manager                                        │
│      Fix: Create departments table with manager                             │
│                                                                              │
│   5. CALCULATED FIELDS                                                       │
│      order_id → subtotal → tax_rate → tax_amount                           │
│      Fix: Store only base values, calculate on read                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

| Concept | Description |
|---------|-------------|
| **3NF Requirement** | No transitive dependencies |
| **Transitive Dependency** | Non-key → non-key dependency |
| **Detection** | Look for non-key attributes that determine others |
| **Fix** | Extract dependent attributes to new table |
| **Motto** | "Depends on the key, the whole key, and nothing but the key" |

**Key Rule**: Every non-key attribute must depend directly on the primary key, not through other non-key attributes.

3NF is the most commonly targeted normal form for production databases - it balances data integrity with practical query performance.
