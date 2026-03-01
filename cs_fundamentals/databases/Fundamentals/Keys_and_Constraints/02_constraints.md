# Database Constraints

## 1. Introduction

**Constraints** are rules enforced by the database to maintain data integrity. They prevent invalid data from being inserted, updated, or deleted.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     CONSTRAINT TYPES                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   DOMAIN CONSTRAINTS        - Valid values for a column                    │
│   • NOT NULL                                                               │
│   • CHECK                                                                  │
│   • DEFAULT                                                                │
│                                                                              │
│   KEY CONSTRAINTS           - Uniqueness and identification                │
│   • PRIMARY KEY                                                            │
│   • UNIQUE                                                                 │
│                                                                              │
│   REFERENTIAL CONSTRAINTS   - Relationships between tables                 │
│   • FOREIGN KEY                                                            │
│                                                                              │
│   ENTITY CONSTRAINTS        - Table-level rules                            │
│   • CHECK (multi-column)                                                   │
│   • EXCLUDE                                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. NOT NULL Constraint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       NOT NULL                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Ensures a column cannot contain NULL values                              │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100) NOT NULL,       -- Required                        │
│       email VARCHAR(255) NOT NULL,      -- Required                        │
│       phone VARCHAR(20),                 -- Optional (NULL allowed)        │
│       department_id INT                  -- Optional                       │
│   );                                                                       │
│                                                                              │
│   -- Adding NOT NULL to existing column                                    │
│   ALTER TABLE employees                                                    │
│   ALTER COLUMN phone SET NOT NULL;                                         │
│                                                                              │
│   -- Removing NOT NULL                                                     │
│   ALTER TABLE employees                                                    │
│   ALTER COLUMN phone DROP NOT NULL;                                        │
│                                                                              │
│   BEST PRACTICES:                                                          │
│   • Use NOT NULL unless there's a valid reason for NULL                   │
│   • Consider defaults instead of allowing NULL                             │
│   • NULL can cause unexpected query results                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. DEFAULT Constraint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DEFAULT                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Provides a default value when none is specified during INSERT            │
│                                                                              │
│   CREATE TABLE orders (                                                     │
│       id SERIAL PRIMARY KEY,                                               │
│       customer_id INT NOT NULL,                                            │
│       status VARCHAR(20) DEFAULT 'pending',                                │
│       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,                      │
│       priority INT DEFAULT 1,                                              │
│       is_active BOOLEAN DEFAULT TRUE                                       │
│   );                                                                       │
│                                                                              │
│   -- Using expressions as defaults                                         │
│   CREATE TABLE products (                                                  │
│       id UUID DEFAULT gen_random_uuid(),                                   │
│       name VARCHAR(100),                                                   │
│       created_by VARCHAR(50) DEFAULT CURRENT_USER,                         │
│       valid_until DATE DEFAULT CURRENT_DATE + INTERVAL '1 year'           │
│   );                                                                       │
│                                                                              │
│   -- Adding/modifying default                                              │
│   ALTER TABLE orders                                                       │
│   ALTER COLUMN status SET DEFAULT 'new';                                   │
│                                                                              │
│   -- Dropping default                                                       │
│   ALTER TABLE orders                                                       │
│   ALTER COLUMN status DROP DEFAULT;                                        │
│                                                                              │
│   NOTE: INSERT with explicit NULL ignores default:                         │
│   INSERT INTO orders (customer_id, status) VALUES (1, NULL);               │
│   -- status will be NULL, not 'pending'                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. UNIQUE Constraint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        UNIQUE                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Ensures all values in a column (or combination) are distinct             │
│   Allows NULL values (multiple NULLs allowed by SQL standard)             │
│                                                                              │
│   -- Single column unique                                                   │
│   CREATE TABLE users (                                                      │
│       id SERIAL PRIMARY KEY,                                               │
│       email VARCHAR(255) UNIQUE,                                           │
│       username VARCHAR(50) UNIQUE NOT NULL                                 │
│   );                                                                       │
│                                                                              │
│   -- Multi-column unique (composite)                                        │
│   CREATE TABLE subscriptions (                                             │
│       id SERIAL PRIMARY KEY,                                               │
│       user_id INT,                                                         │
│       service_id INT,                                                      │
│       UNIQUE (user_id, service_id)  -- User can't subscribe twice         │
│   );                                                                       │
│                                                                              │
│   -- Named unique constraint                                                │
│   CREATE TABLE products (                                                  │
│       id SERIAL PRIMARY KEY,                                               │
│       sku VARCHAR(50),                                                     │
│       CONSTRAINT uq_product_sku UNIQUE (sku)                               │
│   );                                                                       │
│                                                                              │
│   -- Adding unique to existing table                                        │
│   ALTER TABLE users ADD CONSTRAINT uq_phone UNIQUE (phone);               │
│                                                                              │
│   -- PostgreSQL: UNIQUE with NULLS NOT DISTINCT (15+)                      │
│   CREATE UNIQUE INDEX idx_email ON users (email)                           │
│   WHERE email IS NOT NULL;  -- Partial unique (old way)                   │
│                                                                              │
│   UNIQUE (email) NULLS NOT DISTINCT;  -- Only one NULL allowed           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. CHECK Constraint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CHECK                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Ensures values meet specified conditions                                 │
│                                                                              │
│   -- Single column checks                                                   │
│   CREATE TABLE products (                                                  │
│       id SERIAL PRIMARY KEY,                                               │
│       name VARCHAR(100) NOT NULL,                                          │
│       price DECIMAL(10,2) CHECK (price > 0),                              │
│       quantity INT CHECK (quantity >= 0),                                  │
│       discount DECIMAL(5,2) CHECK (discount BETWEEN 0 AND 100),           │
│       status VARCHAR(20) CHECK (status IN ('active', 'inactive', 'draft'))│
│   );                                                                       │
│                                                                              │
│   -- Multi-column checks                                                    │
│   CREATE TABLE events (                                                    │
│       id SERIAL PRIMARY KEY,                                               │
│       name VARCHAR(200),                                                   │
│       start_date DATE,                                                     │
│       end_date DATE,                                                       │
│       CHECK (end_date >= start_date)  -- End must be after start         │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE orders (                                                    │
│       id SERIAL PRIMARY KEY,                                               │
│       subtotal DECIMAL(10,2),                                              │
│       tax DECIMAL(10,2),                                                   │
│       total DECIMAL(10,2),                                                 │
│       CHECK (total = subtotal + tax)  -- Calculated field validation     │
│   );                                                                       │
│                                                                              │
│   -- Named check constraint                                                 │
│   CREATE TABLE employees (                                                  │
│       id SERIAL PRIMARY KEY,                                               │
│       age INT,                                                             │
│       salary DECIMAL(10,2),                                                │
│       CONSTRAINT chk_age CHECK (age BETWEEN 18 AND 120),                  │
│       CONSTRAINT chk_salary CHECK (salary > 0)                            │
│   );                                                                       │
│                                                                              │
│   -- Adding check to existing table                                        │
│   ALTER TABLE products                                                     │
│   ADD CONSTRAINT chk_price_positive CHECK (price > 0);                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. PRIMARY KEY Constraint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PRIMARY KEY                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Combines UNIQUE and NOT NULL                                             │
│   Only one PRIMARY KEY per table                                           │
│                                                                              │
│   -- Single column primary key                                              │
│   CREATE TABLE users (                                                      │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   -- With auto-generation                                                   │
│   CREATE TABLE users (                                                      │
│       id SERIAL PRIMARY KEY,       -- PostgreSQL                          │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE users (                                                      │
│       id INT AUTO_INCREMENT PRIMARY KEY,  -- MySQL                        │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   -- Composite primary key                                                  │
│   CREATE TABLE order_items (                                               │
│       order_id INT,                                                        │
│       line_number INT,                                                     │
│       product_id INT,                                                      │
│       quantity INT,                                                        │
│       PRIMARY KEY (order_id, line_number)                                  │
│   );                                                                       │
│                                                                              │
│   -- Named primary key constraint                                          │
│   CREATE TABLE products (                                                  │
│       id INT,                                                              │
│       CONSTRAINT pk_products PRIMARY KEY (id)                              │
│   );                                                                       │
│                                                                              │
│   -- Adding primary key to existing table                                  │
│   ALTER TABLE existing_table                                               │
│   ADD CONSTRAINT pk_existing PRIMARY KEY (id);                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. FOREIGN KEY Constraint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       FOREIGN KEY                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Maintains referential integrity between tables                           │
│                                                                              │
│   -- Basic foreign key                                                      │
│   CREATE TABLE orders (                                                    │
│       id SERIAL PRIMARY KEY,                                               │
│       customer_id INT REFERENCES customers(id),                            │
│       order_date DATE                                                      │
│   );                                                                       │
│                                                                              │
│   -- With referential actions                                               │
│   CREATE TABLE orders (                                                    │
│       id SERIAL PRIMARY KEY,                                               │
│       customer_id INT,                                                     │
│       FOREIGN KEY (customer_id) REFERENCES customers(id)                   │
│           ON DELETE CASCADE                                                │
│           ON UPDATE CASCADE                                                │
│   );                                                                       │
│                                                                              │
│   -- Composite foreign key                                                  │
│   CREATE TABLE order_details (                                             │
│       id SERIAL PRIMARY KEY,                                               │
│       order_id INT,                                                        │
│       line_number INT,                                                     │
│       notes TEXT,                                                          │
│       FOREIGN KEY (order_id, line_number)                                  │
│           REFERENCES order_items(order_id, line_number)                    │
│   );                                                                       │
│                                                                              │
│   -- Self-referencing foreign key                                          │
│   CREATE TABLE employees (                                                  │
│       id SERIAL PRIMARY KEY,                                               │
│       name VARCHAR(100),                                                   │
│       manager_id INT REFERENCES employees(id)                              │
│   );                                                                       │
│                                                                              │
│   REFERENTIAL ACTIONS:                                                      │
│   • CASCADE    - Propagate changes to child rows                          │
│   • SET NULL   - Set FK to NULL when parent deleted                       │
│   • SET DEFAULT - Set FK to default value                                 │
│   • RESTRICT  - Prevent operation if children exist                       │
│   • NO ACTION - Check at statement end (deferred)                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. EXCLUDE Constraint (PostgreSQL)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   EXCLUDE CONSTRAINT                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Prevents overlapping or conflicting values (PostgreSQL specific)         │
│   More powerful than CHECK for complex conditions                          │
│                                                                              │
│   -- Prevent overlapping date ranges                                        │
│   CREATE TABLE room_bookings (                                             │
│       id SERIAL PRIMARY KEY,                                               │
│       room_id INT,                                                         │
│       during DATERANGE,                                                    │
│       EXCLUDE USING GIST (                                                 │
│           room_id WITH =,                                                  │
│           during WITH &&         -- && means "overlaps"                   │
│       )                                                                    │
│   );                                                                       │
│                                                                              │
│   -- Extension needed for some types                                        │
│   CREATE EXTENSION btree_gist;                                             │
│                                                                              │
│   -- Prevent overlapping time periods for same resource                    │
│   CREATE TABLE reservations (                                              │
│       id SERIAL PRIMARY KEY,                                               │
│       resource_id INT,                                                     │
│       start_time TIMESTAMP,                                                │
│       end_time TIMESTAMP,                                                  │
│       EXCLUDE USING GIST (                                                 │
│           resource_id WITH =,                                              │
│           tsrange(start_time, end_time) WITH &&                           │
│       )                                                                    │
│   );                                                                       │
│                                                                              │
│   -- Prevent adjacent IP ranges                                            │
│   CREATE TABLE network_allocations (                                       │
│       id SERIAL PRIMARY KEY,                                               │
│       network cidr,                                                        │
│       EXCLUDE USING GIST (network inet_ops WITH &&)                       │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Deferred Constraints

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DEFERRED CONSTRAINTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Control WHEN constraints are checked:                                    │
│   • IMMEDIATE - After each statement (default)                            │
│   • DEFERRED  - At transaction commit                                     │
│                                                                              │
│   -- Creating deferrable constraint                                        │
│   CREATE TABLE nodes (                                                     │
│       id INT PRIMARY KEY,                                                  │
│       next_id INT,                                                         │
│       CONSTRAINT fk_next                                                   │
│           FOREIGN KEY (next_id) REFERENCES nodes(id)                       │
│           DEFERRABLE INITIALLY DEFERRED                                    │
│   );                                                                       │
│                                                                              │
│   -- Insert circular references                                             │
│   BEGIN;                                                                   │
│   INSERT INTO nodes (id, next_id) VALUES (1, 2);                          │
│   INSERT INTO nodes (id, next_id) VALUES (2, 1);                          │
│   COMMIT;  -- Constraint checked here, both exist, OK                     │
│                                                                              │
│   -- Deferrable initially immediate                                        │
│   CREATE TABLE t (                                                         │
│       id INT PRIMARY KEY,                                                  │
│       ref_id INT,                                                          │
│       CONSTRAINT fk_ref FOREIGN KEY (ref_id) REFERENCES t(id)             │
│           DEFERRABLE INITIALLY IMMEDIATE                                   │
│   );                                                                       │
│                                                                              │
│   -- Changing defer mode in transaction                                    │
│   BEGIN;                                                                   │
│   SET CONSTRAINTS fk_ref DEFERRED;                                        │
│   -- ... operations ...                                                    │
│   COMMIT;                                                                  │
│                                                                              │
│   SET CONSTRAINTS ALL DEFERRED;    -- All deferrable constraints          │
│   SET CONSTRAINTS ALL IMMEDIATE;   -- Check all now                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Managing Constraints

```sql
-- View all constraints on a table (PostgreSQL)
SELECT
    conname AS constraint_name,
    contype AS type,
    pg_get_constraintdef(oid) AS definition
FROM pg_constraint
WHERE conrelid = 'products'::regclass;

-- Constraint types: p=primary key, u=unique, f=foreign key, c=check, x=exclude

-- Drop constraint by name
ALTER TABLE products DROP CONSTRAINT chk_price_positive;

-- Temporarily disable constraint (PostgreSQL)
ALTER TABLE orders DISABLE TRIGGER ALL;  -- Disables FK checks
-- ... bulk operations ...
ALTER TABLE orders ENABLE TRIGGER ALL;

-- MySQL: Disable foreign key checks
SET FOREIGN_KEY_CHECKS = 0;
-- ... operations ...
SET FOREIGN_KEY_CHECKS = 1;

-- Validate constraint without enforcing (PostgreSQL)
ALTER TABLE products ADD CONSTRAINT chk_valid CHECK (price > 0) NOT VALID;
-- Later, validate existing data:
ALTER TABLE products VALIDATE CONSTRAINT chk_valid;
```

---

## 11. Summary

| Constraint | Purpose | Example |
|------------|---------|---------|
| NOT NULL | Require a value | `name VARCHAR(100) NOT NULL` |
| DEFAULT | Provide default value | `status DEFAULT 'active'` |
| UNIQUE | Ensure distinct values | `email UNIQUE` |
| CHECK | Validate values | `CHECK (age >= 18)` |
| PRIMARY KEY | Unique identifier | `id SERIAL PRIMARY KEY` |
| FOREIGN KEY | Referential integrity | `REFERENCES users(id)` |
| EXCLUDE | Prevent overlaps | `EXCLUDE USING GIST (...)` |

**Best Practices:**
1. Use constraints to enforce business rules in the database
2. Name constraints for easier management
3. Consider deferred constraints for complex operations
4. Use CHECK for domain validation
5. Always define foreign keys for relationships
6. Use NOT NULL by default, allow NULL only when needed
