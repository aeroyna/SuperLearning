# First Normal Form (1NF)

## 1. Introduction

**First Normal Form (1NF)** is the most basic level of database normalization. It establishes the fundamental requirements for a relational database table.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        1NF REQUIREMENTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A table is in 1NF if:                                                     │
│                                                                              │
│   1. Each column contains only ATOMIC (indivisible) values                  │
│   2. No REPEATING GROUPS (no arrays/lists in cells)                         │
│   3. Each row is UNIQUE (has a primary key)                                 │
│   4. Column order doesn't matter                                            │
│   5. Row order doesn't matter                                               │
│                                                                              │
│   Essentially: "Each cell contains exactly one value"                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Violations of 1NF

### 2.1 Non-Atomic Values

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VIOLATION: NON-ATOMIC VALUES                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   WRONG (violates 1NF):                                                     │
│   ┌────────┬─────────┬────────────────────────────┐                        │
│   │ emp_id │  name   │         phone_numbers       │                        │
│   ├────────┼─────────┼────────────────────────────┤                        │
│   │  101   │  Alice  │  555-1234, 555-5678        │  ← Multiple values!    │
│   │  102   │   Bob   │  555-9999                   │                        │
│   │  103   │  Carol  │  555-1111, 555-2222, 555-3333 │                      │
│   └────────┴─────────┴────────────────────────────┘                        │
│                                                                              │
│   Problems:                                                                  │
│   • How do you search for a specific phone?                                 │
│   • How do you count phones per employee?                                   │
│   • How do you delete just one phone?                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Repeating Groups

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VIOLATION: REPEATING GROUPS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   WRONG (violates 1NF):                                                     │
│   ┌──────────┬─────────┬─────────┬─────────┬─────────┬─────────┐           │
│   │ order_id │ prod1   │  qty1   │ prod2   │  qty2   │ prod3   │ ...       │
│   ├──────────┼─────────┼─────────┼─────────┼─────────┼─────────┤           │
│   │   1001   │ Widget  │    2    │ Gadget  │    1    │  NULL   │           │
│   │   1002   │ Gizmo   │    5    │  NULL   │  NULL   │  NULL   │           │
│   │   1003   │ Widget  │    1    │ Gizmo   │    3    │ Thing   │           │
│   └──────────┴─────────┴─────────┴─────────┴─────────┴─────────┘           │
│                                                                              │
│   Problems:                                                                  │
│   • How many product columns do we need?                                    │
│   • Lots of NULLs waste space                                              │
│   • Difficult to query: WHERE prod1 = 'Widget' OR prod2 = 'Widget' OR...   │
│   • Adding more products requires schema change                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Composite Values

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VIOLATION: COMPOSITE VALUES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   WRONG (violates 1NF - debatable):                                         │
│   ┌────────┬───────────────────────────────────┐                           │
│   │ emp_id │            full_address            │                           │
│   ├────────┼───────────────────────────────────┤                           │
│   │  101   │ 123 Main St, New York, NY 10001   │                           │
│   │  102   │ 456 Oak Ave, Boston, MA 02101     │                           │
│   └────────┴───────────────────────────────────┘                           │
│                                                                              │
│   Problems:                                                                  │
│   • Can't easily query by city or state                                     │
│   • Can't sort by zip code                                                  │
│   • Inconsistent formatting possible                                        │
│                                                                              │
│   However, if you NEVER need to query parts separately,                     │
│   some consider this acceptable (context-dependent)                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Converting to 1NF

### 3.1 Fix Non-Atomic Values

```sql
-- BEFORE (non-1NF): Multiple phones in one column
-- employees(emp_id, name, phone_numbers)

-- AFTER (1NF): Separate table for phones
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employee_phones (
    emp_id INT,
    phone VARCHAR(20),
    phone_type VARCHAR(20),  -- 'mobile', 'home', 'work'
    PRIMARY KEY (emp_id, phone),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id)
);

-- Insert data
INSERT INTO employees VALUES (101, 'Alice');
INSERT INTO employee_phones VALUES
    (101, '555-1234', 'mobile'),
    (101, '555-5678', 'home');

-- Now queries are easy:
SELECT * FROM employee_phones WHERE phone = '555-1234';
SELECT emp_id, COUNT(*) FROM employee_phones GROUP BY emp_id;
```

### 3.2 Fix Repeating Groups

```sql
-- BEFORE (non-1NF): Repeating product columns
-- orders(order_id, prod1, qty1, prod2, qty2, prod3, qty3, ...)

-- AFTER (1NF): Separate table for order items
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

-- Insert data
INSERT INTO orders VALUES (1001, 500, '2024-01-15');
INSERT INTO order_items VALUES
    (1001, 101, 2, 19.99),  -- 2 Widgets
    (1001, 102, 1, 29.99);  -- 1 Gadget

-- Flexible: any number of items per order
-- Easy to query: SELECT * FROM order_items WHERE product_id = 101
```

### 3.3 Fix Composite Values

```sql
-- BEFORE (non-1NF): Address as single field
-- employees(emp_id, name, full_address)

-- AFTER (1NF): Address split into components
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    street VARCHAR(255),
    city VARCHAR(100),
    state CHAR(2),
    zip_code VARCHAR(10),
    country VARCHAR(50) DEFAULT 'USA'
);

-- Now we can:
SELECT * FROM employees WHERE city = 'New York';
SELECT * FROM employees WHERE state = 'NY';
SELECT * FROM employees ORDER BY zip_code;
SELECT state, COUNT(*) FROM employees GROUP BY state;
```

---

## 4. Edge Cases and Considerations

### 4.1 JSON Columns

```sql
-- Modern databases support JSON - is this 1NF?

CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB  -- {"color": "red", "size": "L", "weight": 2.5}
);

-- Arguments FOR allowing JSON:
-- • Flexible schema for varying attributes
-- • Modern databases index JSON fields
-- • Query support: WHERE attributes->>'color' = 'red'

-- Arguments AGAINST:
-- • Not truly atomic if you query inside the JSON
-- • Can hide repeating groups
-- • Makes joins and aggregations harder

-- GUIDELINE: Use JSON for truly unstructured/dynamic data,
-- not to avoid proper normalization
```

### 4.2 Arrays (PostgreSQL)

```sql
-- PostgreSQL supports array types
CREATE TABLE posts (
    id INT PRIMARY KEY,
    title VARCHAR(200),
    tags TEXT[]  -- {'tech', 'database', 'sql'}
);

-- Technically violates 1NF, but...
-- • PostgreSQL can index arrays: CREATE INDEX ON posts USING GIN(tags);
-- • Query support: WHERE 'database' = ANY(tags)
-- • Often more practical than a junction table

-- When to use:
-- • Fixed set of simple values
-- • Rarely queried individually
-- • Performance testing shows it's faster

-- When to normalize:
-- • Values have their own attributes (tag name, tag color, etc.)
-- • Need referential integrity
-- • Frequently filtered or joined on
```

### 4.3 The "Name" Debate

```sql
-- Is this a 1NF violation?
CREATE TABLE employees (
    id INT PRIMARY KEY,
    full_name VARCHAR(100)  -- "John Michael Smith"
);

-- vs

CREATE TABLE employees (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    middle_name VARCHAR(50),
    last_name VARCHAR(50)
);

-- It depends on your use case:
-- • If you NEVER search/sort by first name: single column is fine
-- • If you need to address people as "Mr. Smith": split it
-- • Cultural consideration: not all names fit Western patterns

-- Pragmatic approach: Split if you'll query parts separately
```

---

## 5. Benefits of 1NF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      BENEFITS OF 1NF                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   QUERY SIMPLICITY                                                          │
│   • Simple WHERE clauses: WHERE phone = '555-1234'                          │
│   • Easy aggregations: COUNT, SUM, AVG work naturally                       │
│   • Standard joins work as expected                                         │
│                                                                              │
│   DATA INTEGRITY                                                             │
│   • Constraints can be applied per value                                    │
│   • Foreign keys work properly                                              │
│   • Uniqueness can be enforced                                              │
│                                                                              │
│   FLEXIBILITY                                                                │
│   • Add more values without schema changes                                  │
│   • Modify individual values easily                                         │
│   • Delete specific values without affecting others                         │
│                                                                              │
│   PERFORMANCE                                                                │
│   • Indexes work effectively on atomic columns                              │
│   • No need to parse strings to find values                                 │
│   • Optimizer can use statistics properly                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Example: Complete 1NF Transformation

### Before (Not 1NF)

```
┌──────────┬─────────┬──────────────────────────┬────────────────────────────┐
│ order_id │customer │ products                 │ delivery_address           │
├──────────┼─────────┼──────────────────────────┼────────────────────────────┤
│   1001   │  Alice  │ Widget(2), Gadget(1)     │ 123 Main St, NYC, NY 10001│
│   1002   │   Bob   │ Gizmo(5)                 │ 456 Oak Ave, Boston, MA    │
│   1003   │  Alice  │ Widget(1), Gizmo(3),     │ 123 Main St, NYC, NY 10001│
│          │         │ Thing(2)                 │                            │
└──────────┴─────────┴──────────────────────────┴────────────────────────────┘

Violations:
1. products: multiple values, repeating group
2. delivery_address: composite value
```

### After (1NF)

```sql
-- Customers table
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

-- Addresses table
CREATE TABLE addresses (
    address_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    street VARCHAR(255),
    city VARCHAR(100),
    state CHAR(2),
    zip_code VARCHAR(10)
);

-- Products table
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

-- Orders table
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    delivery_address_id INT REFERENCES addresses(address_id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Order Items table (eliminates repeating group)
CREATE TABLE order_items (
    order_id INT REFERENCES orders(order_id),
    product_id INT REFERENCES products(product_id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

---

## 7. Summary

| Rule | Violation Example | Fix |
|------|-------------------|-----|
| Atomic values | "555-1234, 555-5678" | Separate table for phones |
| No repeating groups | prod1, qty1, prod2, qty2... | Separate table for items |
| Unique rows | Duplicate rows | Add primary key |
| Single value per cell | "red, blue, green" | Separate table or proper array |

**Key Principle**: Each cell should contain exactly one value, and each row should be uniquely identifiable.

1NF is the foundation - without it, the database isn't truly relational and higher normal forms are impossible to achieve.
