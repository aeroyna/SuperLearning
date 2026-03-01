# Key Types in Databases

## 1. Introduction

**Keys** are essential database concepts that uniquely identify rows and establish relationships between tables. Understanding different key types is fundamental to proper database design.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          KEY TYPES OVERVIEW                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   IDENTIFICATION KEYS:                                                      │
│   • Super Key     - Any set of columns that uniquely identifies rows       │
│   • Candidate Key - Minimal super key (no redundant columns)               │
│   • Primary Key   - Chosen candidate key for the table                     │
│   • Alternate Key - Candidate keys not chosen as primary                   │
│                                                                              │
│   RELATIONSHIP KEYS:                                                        │
│   • Foreign Key   - References primary key in another table                │
│   • Composite Key - Multiple columns forming a key                         │
│                                                                              │
│   SPECIAL KEYS:                                                             │
│   • Natural Key   - Business-meaningful identifier                         │
│   • Surrogate Key - System-generated identifier                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Super Key

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SUPER KEY                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Definition: Any set of columns that can uniquely identify a row          │
│   A super key may have extra (redundant) columns                           │
│                                                                              │
│   EMPLOYEE table:                                                           │
│   ┌─────┬───────────┬─────────────────────┬────────────┐                   │
│   │ id  │   name    │       email         │   dept_id  │                   │
│   ├─────┼───────────┼─────────────────────┼────────────┤                   │
│   │  1  │  Alice    │ alice@company.com   │     10     │                   │
│   │  2  │   Bob     │  bob@company.com    │     20     │                   │
│   │  3  │  Alice    │ alice2@company.com  │     10     │                   │
│   └─────┴───────────┴─────────────────────┴────────────┘                   │
│                                                                              │
│   Super Keys:                                                               │
│   • {id}                           ← Minimal (candidate key)               │
│   • {email}                        ← Minimal (candidate key)               │
│   • {id, name}                     ← Has redundant column                  │
│   • {id, email}                    ← Has redundant column                  │
│   • {id, name, email}              ← Has redundant columns                 │
│   • {id, name, email, dept_id}     ← All columns                          │
│                                                                              │
│   NOT super keys (don't uniquely identify):                                │
│   • {name}         ← Two employees named "Alice"                          │
│   • {dept_id}      ← Multiple employees per department                    │
│   • {name, dept_id} ← Still not unique                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Candidate Key

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CANDIDATE KEY                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Definition: A MINIMAL super key                                           │
│   No subset of a candidate key is itself a super key                       │
│   All candidate keys could serve as the primary key                        │
│                                                                              │
│   From previous example:                                                    │
│   Candidate Keys: {id}, {email}                                            │
│                                                                              │
│   NOT candidate keys:                                                       │
│   • {id, name}  ← Can remove 'name', still unique                         │
│   • {email, id} ← Can remove either, still unique                         │
│                                                                              │
│   Finding candidate keys:                                                   │
│   1. Start with columns that appear to be unique                           │
│   2. Verify uniqueness with data analysis                                  │
│   3. Check if any subset is also unique (if so, it's not minimal)         │
│                                                                              │
│   SQL to check for candidate key:                                          │
│   SELECT email, COUNT(*) FROM employees                                    │
│   GROUP BY email                                                           │
│   HAVING COUNT(*) > 1;                                                     │
│   -- If returns rows, email is NOT a candidate key                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Primary Key

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRIMARY KEY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Definition: The chosen candidate key to uniquely identify rows           │
│   Each table should have exactly one primary key                           │
│                                                                              │
│   PROPERTIES:                                                               │
│   • Unique: No two rows have same PK value                                 │
│   • Not Null: Every row must have a PK value                              │
│   • Immutable: Should not change (best practice)                          │
│   • Stable: Value has no business meaning that might change               │
│                                                                              │
│   SQL:                                                                      │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,          -- Single column PK                     │
│       email VARCHAR(255) UNIQUE,   -- Alternate key                        │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   -- Composite primary key                                                  │
│   CREATE TABLE order_items (                                               │
│       order_id INT,                                                        │
│       product_id INT,                                                      │
│       quantity INT,                                                        │
│       PRIMARY KEY (order_id, product_id)                                   │
│   );                                                                       │
│                                                                              │
│   -- Named primary key constraint                                          │
│   CREATE TABLE products (                                                  │
│       id INT,                                                              │
│       name VARCHAR(100),                                                   │
│       CONSTRAINT pk_products PRIMARY KEY (id)                              │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Alternate Key (Unique Key)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       ALTERNATE KEY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Definition: Candidate keys that were NOT chosen as primary key           │
│   Also called "unique keys"                                                │
│                                                                              │
│   If we chose 'id' as PK, then 'email' is an alternate key:               │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                  -- Primary key                  │
│       email VARCHAR(255) UNIQUE NOT NULL,  -- Alternate key               │
│       ssn CHAR(11) UNIQUE,                 -- Another alternate key       │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   DIFFERENCES FROM PRIMARY KEY:                                            │
│   • Can allow NULL values (unless explicitly NOT NULL)                    │
│   • Table can have multiple alternate keys                                 │
│   • Often used for natural identifiers                                    │
│                                                                              │
│   USE CASES:                                                                │
│   • email (natural unique identifier)                                      │
│   • ssn (government-issued ID)                                            │
│   • product_sku (business identifier)                                     │
│   • isbn (book identifier)                                                │
│                                                                              │
│   -- Composite unique constraint                                            │
│   ALTER TABLE orders ADD CONSTRAINT uq_customer_order                      │
│       UNIQUE (customer_id, order_date);                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Foreign Key

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FOREIGN KEY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Definition: Column(s) that reference the primary key of another table    │
│   Establishes relationships between tables                                 │
│                                                                              │
│   ┌───────────────┐              ┌───────────────┐                         │
│   │  departments  │              │   employees   │                         │
│   ├───────────────┤              ├───────────────┤                         │
│   │ id (PK)       │◄─────────────│ dept_id (FK)  │                         │
│   │ name          │              │ id (PK)       │                         │
│   └───────────────┘              │ name          │                         │
│                                  └───────────────┘                         │
│                                                                              │
│   SQL:                                                                      │
│   CREATE TABLE departments (                                               │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       dept_id INT REFERENCES departments(id)                               │
│   );                                                                       │
│                                                                              │
│   -- With explicit constraint name and actions                             │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       dept_id INT,                                                         │
│       CONSTRAINT fk_dept FOREIGN KEY (dept_id)                            │
│           REFERENCES departments(id)                                       │
│           ON DELETE SET NULL                                               │
│           ON UPDATE CASCADE                                                │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.1 Foreign Key Actions

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FOREIGN KEY ACTIONS                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ON DELETE actions (what happens when referenced row is deleted):         │
│                                                                              │
│   CASCADE     - Delete the referencing rows too                            │
│   SET NULL    - Set FK to NULL                                             │
│   SET DEFAULT - Set FK to default value                                    │
│   RESTRICT    - Prevent delete if referenced (default in some DBs)        │
│   NO ACTION   - Check at end of statement (default in SQL standard)       │
│                                                                              │
│   ON UPDATE actions (what happens when referenced PK changes):             │
│   (Same options as ON DELETE)                                              │
│                                                                              │
│   EXAMPLES:                                                                 │
│                                                                              │
│   -- Delete all orders when customer is deleted                            │
│   FOREIGN KEY (customer_id) REFERENCES customers(id)                       │
│       ON DELETE CASCADE                                                    │
│                                                                              │
│   -- Set employee's dept to NULL when department is deleted               │
│   FOREIGN KEY (dept_id) REFERENCES departments(id)                         │
│       ON DELETE SET NULL                                                   │
│                                                                              │
│   -- Prevent deleting category if products exist                          │
│   FOREIGN KEY (category_id) REFERENCES categories(id)                      │
│       ON DELETE RESTRICT                                                   │
│                                                                              │
│   -- Update FK when parent PK changes                                      │
│   FOREIGN KEY (manager_id) REFERENCES employees(id)                        │
│       ON UPDATE CASCADE                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Natural vs Surrogate Keys

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   NATURAL vs SURROGATE KEYS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   NATURAL KEY                        SURROGATE KEY                          │
│   ───────────                        ─────────────                          │
│   Business-meaningful                System-generated                       │
│   Already exists in data             Created specifically as identifier     │
│                                                                              │
│   Examples:                          Examples:                              │
│   • SSN                              • Auto-increment INT                   │
│   • Email                            • UUID                                 │
│   • ISBN                             • SERIAL                               │
│   • Product SKU                      • IDENTITY                             │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │ NATURAL KEY                 │  SURROGATE KEY                    │      │
│   ├─────────────────────────────┼───────────────────────────────────┤      │
│   │ ✓ Meaningful                │  ✓ Simple and stable              │      │
│   │ ✓ Already unique            │  ✓ Never changes                  │      │
│   │ ✓ No extra column needed    │  ✓ Consistent size                │      │
│   │ ✗ May change over time      │  ✓ No business dependency        │      │
│   │ ✗ Can be complex (multi-col)│  ✗ Extra storage                  │      │
│   │ ✗ Size varies (varchar)     │  ✗ Meaningless to users           │      │
│   │ ✗ Privacy concerns (SSN)    │  ✗ Joins may be slower            │      │
│   └─────────────────────────────┴───────────────────────────────────┘      │
│                                                                              │
│   BEST PRACTICE:                                                           │
│   • Use surrogate PK for most tables                                       │
│   • Keep natural key as UNIQUE constraint                                  │
│   • Use natural key for lookup/display                                     │
│                                                                              │
│   CREATE TABLE products (                                                  │
│       id SERIAL PRIMARY KEY,           -- Surrogate key                   │
│       sku VARCHAR(50) UNIQUE NOT NULL, -- Natural key (alternate)         │
│       name VARCHAR(200)                                                    │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Composite Keys

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       COMPOSITE KEYS                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Definition: A key composed of multiple columns                           │
│   Individual columns may not be unique, but combination is                 │
│                                                                              │
│   USE CASES:                                                                │
│   • Junction tables (M:N relationships)                                    │
│   • Weak entities                                                          │
│   • When no single column is unique                                        │
│                                                                              │
│   EXAMPLE 1: Junction table                                                │
│   CREATE TABLE student_courses (                                           │
│       student_id INT REFERENCES students(id),                              │
│       course_id INT REFERENCES courses(id),                                │
│       semester VARCHAR(20),                                                │
│       grade CHAR(2),                                                       │
│       PRIMARY KEY (student_id, course_id, semester)                        │
│   );                                                                       │
│                                                                              │
│   EXAMPLE 2: Weak entity                                                   │
│   CREATE TABLE order_items (                                               │
│       order_id INT REFERENCES orders(id),                                  │
│       line_number INT,  -- Unique within order                            │
│       product_id INT,                                                      │
│       quantity INT,                                                        │
│       PRIMARY KEY (order_id, line_number)                                  │
│   );                                                                       │
│                                                                              │
│   CONSIDERATIONS:                                                           │
│   • Order of columns matters for indexing                                  │
│   • All columns must be NOT NULL                                          │
│   • Can be referenced by composite FKs                                    │
│   • May prefer surrogate key if composite is complex                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

| Key Type | Definition | Example |
|----------|------------|---------|
| Super Key | Any unique identifier (may have extras) | {id, name} |
| Candidate Key | Minimal super key | {id} or {email} |
| Primary Key | Chosen candidate key | id |
| Alternate Key | Unused candidate keys | email, ssn |
| Foreign Key | References another table's PK | dept_id → departments.id |
| Natural Key | Business-meaningful | SSN, ISBN |
| Surrogate Key | System-generated | SERIAL, UUID |
| Composite Key | Multiple columns | (order_id, product_id) |

**Key Selection Guidelines:**
1. Choose stable, simple primary keys
2. Prefer surrogate keys for flexibility
3. Keep natural keys as UNIQUE constraints
4. Use composite keys only when necessary
5. Define foreign keys for all relationships
6. Consider performance implications
