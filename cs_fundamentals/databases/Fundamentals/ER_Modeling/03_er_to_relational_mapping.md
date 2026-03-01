# ER to Relational Mapping

## 1. Introduction

Converting an Entity-Relationship (ER) model to a relational database schema follows systematic rules. This process transforms conceptual design into implementable tables.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ER TO RELATIONAL MAPPING                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER Model (Conceptual)          Relational Model (Logical)                │
│   ─────────────────────          ─────────────────────────                  │
│   Entity Type            →       Table (Relation)                          │
│   Attribute              →       Column (Field)                            │
│   Primary Key            →       Primary Key                               │
│   Relationship           →       Foreign Key / Junction Table              │
│                                                                              │
│   MAPPING STEPS:                                                           │
│   1. Map strong entities                                                   │
│   2. Map weak entities                                                     │
│   3. Map 1:1 relationships                                                 │
│   4. Map 1:N relationships                                                 │
│   5. Map M:N relationships                                                 │
│   6. Map multi-valued attributes                                           │
│   7. Map composite attributes                                              │
│   8. Map higher-degree relationships                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Mapping Strong Entities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MAPPING STRONG ENTITIES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER ENTITY:                        RELATIONAL TABLE:                       │
│   ┌─────────────────┐               CREATE TABLE employees (               │
│   │    EMPLOYEE     │                   employee_id INT PRIMARY KEY,       │
│   ├─────────────────┤      →            first_name VARCHAR(50),            │
│   │ employee_id     │                   last_name VARCHAR(50),             │
│   │ first_name      │                   hire_date DATE,                    │
│   │ last_name       │                   salary DECIMAL(10,2)               │
│   │ hire_date       │               );                                      │
│   │ salary          │                                                       │
│   └─────────────────┘                                                       │
│                                                                              │
│   RULES:                                                                    │
│   • Create a table for each strong entity                                  │
│   • Include all simple attributes as columns                               │
│   • Primary key attribute becomes PRIMARY KEY                              │
│   • Choose appropriate SQL data types                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Mapping Weak Entities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MAPPING WEAK ENTITIES                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER MODEL:                                                                 │
│   ┌─────────────────┐         ╔═════════════════╗                          │
│   │    EMPLOYEE     │─────────║   DEPENDENT     ║                          │
│   ├─────────────────┤         ╠═════════════════╣                          │
│   │ employee_id(PK) │         ║ name            ║                          │
│   │ name            │         ║ relationship    ║                          │
│   └─────────────────┘         ║ date_of_birth   ║                          │
│                               ╚═════════════════╝                          │
│                                                                              │
│   RELATIONAL TABLES:                                                        │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       employee_id INT PRIMARY KEY,                                         │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE dependents (                                                │
│       employee_id INT REFERENCES employees(employee_id)                    │
│           ON DELETE CASCADE,                     -- Identifying relationship│
│       name VARCHAR(100),                         -- Partial key            │
│       relationship VARCHAR(50),                                            │
│       date_of_birth DATE,                                                  │
│       PRIMARY KEY (employee_id, name)            -- Composite PK           │
│   );                                                                       │
│                                                                              │
│   RULES:                                                                    │
│   • Create table for weak entity                                           │
│   • Include FK to owner entity (with ON DELETE CASCADE)                   │
│   • PK = owner's PK + partial key (discriminator)                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Mapping 1:1 Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   MAPPING 1:1 RELATIONSHIPS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   THREE OPTIONS:                                                            │
│                                                                              │
│   OPTION 1: Foreign Key Approach (preferred)                               │
│   Place FK in table with total participation, or either if both are total │
│                                                                              │
│   ┌──────────┐     manages     ┌────────────┐                              │
│   │ EMPLOYEE │═════════════════│ DEPARTMENT │                              │
│   └──────────┘                 └────────────┘                              │
│      (0..1)                         (1)                                    │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE departments (                                               │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       manager_id INT UNIQUE REFERENCES employees(id)  -- FK here          │
│   );                                                                       │
│                                                                              │
│   OPTION 2: Merged Table                                                   │
│   Merge both entities into one table (when both have total participation) │
│                                                                              │
│   CREATE TABLE employee_with_parking (                                     │
│       employee_id INT PRIMARY KEY,                                         │
│       employee_name VARCHAR(100),                                          │
│       parking_spot VARCHAR(10)  -- Merged from parking table              │
│   );                                                                       │
│                                                                              │
│   OPTION 3: Cross-Reference (Relationship Table)                           │
│   Create separate junction table (rarely needed for 1:1)                   │
│                                                                              │
│   CREATE TABLE manages (                                                   │
│       employee_id INT UNIQUE REFERENCES employees(id),                     │
│       department_id INT UNIQUE REFERENCES departments(id),                 │
│       start_date DATE,                                                     │
│       PRIMARY KEY (employee_id)  -- or department_id                      │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Mapping 1:N Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   MAPPING 1:N RELATIONSHIPS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER MODEL:                                                                 │
│   ┌────────────┐     contains     ┌───────────┐                            │
│   │ DEPARTMENT │══════════════════│ EMPLOYEE  │                            │
│   └────────────┘       1:N        └───────────┘                            │
│        (1)                             (N)                                  │
│                                                                              │
│   MAPPING RULE: Add FK to the "many" side                                  │
│                                                                              │
│   CREATE TABLE departments (                                               │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       hire_date DATE,                                                      │
│       department_id INT REFERENCES departments(id)  -- FK on "many" side  │
│   );                                                                       │
│                                                                              │
│   WITH RELATIONSHIP ATTRIBUTES:                                            │
│   -- If relationship has attributes, they go with the FK                   │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       department_id INT REFERENCES departments(id),                        │
│       assignment_date DATE,    -- Relationship attribute                  │
│       role VARCHAR(50)         -- Relationship attribute                  │
│   );                                                                       │
│                                                                              │
│   PARTICIPATION CONSTRAINTS:                                                │
│   • Total on many side: NOT NULL on FK                                    │
│   • Partial on many side: FK allows NULL                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Mapping M:N Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   MAPPING M:N RELATIONSHIPS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER MODEL:                                                                 │
│   ┌──────────┐     enrolls      ┌───────────┐                              │
│   │ STUDENT  │══════════════════│  COURSE   │                              │
│   └──────────┘       M:N        └───────────┘                              │
│        │                             │                                      │
│        │   grade, semester          │                                      │
│        └────────────────────────────┘                                      │
│                                                                              │
│   MAPPING: Create junction table                                           │
│                                                                              │
│   CREATE TABLE students (                                                   │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       email VARCHAR(255)                                                   │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE courses (                                                    │
│       id INT PRIMARY KEY,                                                  │
│       title VARCHAR(200),                                                  │
│       credits INT                                                          │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE enrollments (        -- Junction table                      │
│       student_id INT REFERENCES students(id),                              │
│       course_id INT REFERENCES courses(id),                                │
│       grade VARCHAR(2),             -- Relationship attribute              │
│       semester VARCHAR(20),         -- Relationship attribute              │
│       enrollment_date DATE,                                                │
│       PRIMARY KEY (student_id, course_id)                                  │
│   );                                                                       │
│                                                                              │
│   ALTERNATIVES FOR PRIMARY KEY:                                            │
│   • Composite PK (student_id, course_id) - if one enrollment per pair    │
│   • Add surrogate key if multiple enrollments allowed                     │
│                                                                              │
│   CREATE TABLE enrollments (                                               │
│       enrollment_id INT PRIMARY KEY,  -- Surrogate key                    │
│       student_id INT REFERENCES students(id),                              │
│       course_id INT REFERENCES courses(id),                                │
│       semester VARCHAR(20),           -- Now can enroll multiple times    │
│       grade VARCHAR(2)                                                     │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Mapping Multi-Valued Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                MAPPING MULTI-VALUED ATTRIBUTES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER MODEL:                                                                 │
│   ┌─────────────────┐                                                       │
│   │    EMPLOYEE     │                                                       │
│   ├─────────────────┤                                                       │
│   │ id              │                                                       │
│   │ name            │                                                       │
│   │ (phone_numbers) │  ← Multi-valued (double ellipse)                     │
│   │ (skills)        │  ← Multi-valued                                      │
│   └─────────────────┘                                                       │
│                                                                              │
│   MAPPING: Create separate table for each multi-valued attribute           │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employee_phones (                                           │
│       employee_id INT REFERENCES employees(id) ON DELETE CASCADE,         │
│       phone VARCHAR(20),                                                   │
│       phone_type VARCHAR(20),  -- 'mobile', 'home', 'work'                │
│       PRIMARY KEY (employee_id, phone)                                     │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employee_skills (                                           │
│       employee_id INT REFERENCES employees(id) ON DELETE CASCADE,         │
│       skill VARCHAR(100),                                                  │
│       proficiency_level VARCHAR(20),  -- 'beginner', 'intermediate', etc. │
│       PRIMARY KEY (employee_id, skill)                                     │
│   );                                                                       │
│                                                                              │
│   ALTERNATIVE (PostgreSQL arrays):                                         │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       phone_numbers TEXT[],     -- Array type                              │
│       skills TEXT[]             -- Array type                              │
│   );                                                                       │
│   -- Less normalized, but simpler queries                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Mapping Composite Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 MAPPING COMPOSITE ATTRIBUTES                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ER MODEL:                                                                 │
│   ┌─────────────────────────┐                                               │
│   │        EMPLOYEE         │                                               │
│   ├─────────────────────────┤                                               │
│   │ id                      │                                               │
│   │ name                    │                                               │
│   │   ├── first_name        │                                               │
│   │   ├── middle_name       │                                               │
│   │   └── last_name         │                                               │
│   │ address                 │                                               │
│   │   ├── street            │                                               │
│   │   ├── city              │                                               │
│   │   ├── state             │                                               │
│   │   └── zip               │                                               │
│   └─────────────────────────┘                                               │
│                                                                              │
│   MAPPING: Flatten to leaf-level components                                │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       -- Composite: name                                                   │
│       first_name VARCHAR(50),                                              │
│       middle_name VARCHAR(50),                                             │
│       last_name VARCHAR(50),                                               │
│       -- Composite: address                                                │
│       street VARCHAR(255),                                                 │
│       city VARCHAR(100),                                                   │
│       state VARCHAR(50),                                                   │
│       zip VARCHAR(20)                                                      │
│   );                                                                       │
│                                                                              │
│   ALTERNATIVE: Use separate address table (if address is reused)          │
│   CREATE TABLE addresses (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       street VARCHAR(255),                                                 │
│       city VARCHAR(100),                                                   │
│       state VARCHAR(50),                                                   │
│       zip VARCHAR(20)                                                      │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       first_name VARCHAR(50),                                              │
│       middle_name VARCHAR(50),                                             │
│       last_name VARCHAR(50),                                               │
│       address_id INT REFERENCES addresses(id)                              │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Complete Mapping Example

```sql
-- ER MODEL: E-commerce system
-- Entities: Customer, Product, Order, Category
-- Relationships:
--   Customer PLACES Order (1:N)
--   Order CONTAINS Product (M:N with quantity, price)
--   Product BELONGS_TO Category (N:1)
--   Product has multi-valued: tags

-- MAPPING RESULT:

CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    -- Composite: address
    street VARCHAR(255),
    city VARCHAR(100),
    state VARCHAR(50),
    zip VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    category_id INT REFERENCES categories(id)  -- 1:N FK
);

-- Multi-valued attribute
CREATE TABLE product_tags (
    product_id INT REFERENCES products(id) ON DELETE CASCADE,
    tag VARCHAR(50),
    PRIMARY KEY (product_id, tag)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL REFERENCES customers(id),  -- 1:N FK
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'pending',
    shipping_address TEXT
);

-- M:N junction table with attributes
CREATE TABLE order_items (
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,  -- Price at time of order
    PRIMARY KEY (order_id, product_id)
);
```

---

## 10. Summary

| ER Construct | Relational Mapping |
|--------------|-------------------|
| Strong Entity | Table with PK |
| Weak Entity | Table with composite PK (owner FK + partial key) |
| 1:1 Relationship | FK with UNIQUE in one table, or merge |
| 1:N Relationship | FK on "many" side |
| M:N Relationship | Junction table with two FKs |
| Multi-valued Attribute | Separate table with FK |
| Composite Attribute | Flatten to simple columns |
| Derived Attribute | Computed column or calculate in query |
| Relationship Attribute | Column in junction table or on FK side |
