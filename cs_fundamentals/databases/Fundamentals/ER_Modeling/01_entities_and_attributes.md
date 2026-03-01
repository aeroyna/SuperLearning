# Entities and Attributes

## 1. Introduction

**Entities** are the core objects in an ER model - the "things" we want to store information about. **Attributes** describe the properties of those entities.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ENTITIES AND ATTRIBUTES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ENTITY: A distinguishable object about which we store information        │
│   Examples: Customer, Product, Order, Employee                              │
│                                                                              │
│   ATTRIBUTE: A property that describes an entity                            │
│   Examples: Name, Price, Date, Salary                                       │
│                                                                              │
│   ┌─────────────────────────────────┐                                       │
│   │          CUSTOMER               │                                       │
│   ├─────────────────────────────────┤                                       │
│   │ • customer_id (key)             │                                       │
│   │ • first_name                    │                                       │
│   │ • last_name                     │                                       │
│   │ • email                         │                                       │
│   │ • phone                         │                                       │
│   │ • date_of_birth                 │                                       │
│   └─────────────────────────────────┘                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Entity Types

### 2.1 Strong Entities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       STRONG ENTITY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Can exist independently                                                   │
│   Has its own unique identifier (primary key)                              │
│   Represented by a rectangle                                                │
│                                                                              │
│   ┌───────────────────────┐                                                 │
│   │       EMPLOYEE        │                                                 │
│   ├───────────────────────┤                                                 │
│   │ employee_id (PK)      │                                                 │
│   │ name                  │                                                 │
│   │ hire_date             │                                                 │
│   └───────────────────────┘                                                 │
│                                                                              │
│   Examples:                                                                 │
│   • Customer (can exist without orders)                                    │
│   • Product (can exist without being ordered)                              │
│   • Department (exists independently)                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Weak Entities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        WEAK ENTITY                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Cannot exist without its parent (strong) entity                          │
│   Has partial key (discriminator) + parent's key                           │
│   Represented by double-lined rectangle                                    │
│                                                                              │
│   ╔═══════════════════════╗                                                 │
│   ║      DEPENDENT        ║                                                 │
│   ╠═══════════════════════╣                                                 │
│   ║ name (partial key)    ║                                                 │
│   ║ relationship          ║                                                 │
│   ║ date_of_birth         ║                                                 │
│   ╚═══════════════════════╝                                                 │
│            ║                                                                 │
│            ║ belongs to                                                     │
│            ▼                                                                 │
│   ┌───────────────────────┐                                                 │
│   │       EMPLOYEE        │                                                 │
│   └───────────────────────┘                                                 │
│                                                                              │
│   Full key = employee_id + dependent_name                                   │
│   Dependent can't exist without an Employee                                │
│                                                                              │
│   Other examples:                                                           │
│   • OrderItem (needs Order)                                                │
│   • Room (needs Building)                                                  │
│   • PaymentInstallment (needs Loan)                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Associative Entities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ASSOCIATIVE ENTITY                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Represents a many-to-many relationship with its own attributes           │
│   Also called "bridge entity" or "junction table"                          │
│                                                                              │
│   ┌──────────────┐         ┌──────────────────┐         ┌─────────────┐    │
│   │   STUDENT    │         │    ENROLLMENT    │         │   COURSE    │    │
│   ├──────────────┤         ├──────────────────┤         ├─────────────┤    │
│   │ student_id   │◄────────│ student_id (FK)  │────────►│ course_id   │    │
│   │ name         │         │ course_id (FK)   │         │ title       │    │
│   │ email        │         │ enrollment_date  │         │ credits     │    │
│   └──────────────┘         │ grade            │         └─────────────┘    │
│                            │ status           │                            │
│                            └──────────────────┘                            │
│                                                                              │
│   ENROLLMENT has its own attributes beyond the relationship               │
│   PK = (student_id, course_id)                                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Attribute Types

### 3.1 Simple vs Composite Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  SIMPLE vs COMPOSITE ATTRIBUTES                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SIMPLE (Atomic): Cannot be divided further                               │
│   • age                                                                    │
│   • employee_id                                                            │
│   • salary                                                                 │
│                                                                              │
│   COMPOSITE: Can be divided into smaller parts                             │
│                                                                              │
│   full_name                         address                                │
│   ├── first_name                    ├── street                             │
│   ├── middle_name                   ├── city                               │
│   └── last_name                     ├── state                              │
│                                     ├── zip_code                           │
│                                     └── country                            │
│                                                                              │
│   SQL Implementation:                                                       │
│   -- Option 1: Composite as separate columns                               │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       first_name VARCHAR(50),                                              │
│       middle_name VARCHAR(50),                                             │
│       last_name VARCHAR(50)                                                │
│   );                                                                       │
│                                                                              │
│   -- Option 2: Single column (when not queried separately)                 │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       full_name VARCHAR(150)                                               │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Single-Valued vs Multi-Valued Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              SINGLE-VALUED vs MULTI-VALUED ATTRIBUTES                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SINGLE-VALUED: Has exactly one value per entity                          │
│   • date_of_birth (one per person)                                        │
│   • ssn (one per person)                                                  │
│   • employee_id                                                            │
│                                                                              │
│   MULTI-VALUED: Can have multiple values per entity                       │
│   • phone_numbers (person can have multiple)                              │
│   • skills (employee can have many)                                       │
│   • email_addresses                                                        │
│                                                                              │
│   ER Diagram notation: Double ellipse for multi-valued                    │
│                                                                              │
│   SQL Implementation for multi-valued:                                     │
│   -- Separate table                                                        │
│   CREATE TABLE employee_phones (                                           │
│       employee_id INT REFERENCES employees(id),                            │
│       phone VARCHAR(20),                                                   │
│       phone_type VARCHAR(20),                                              │
│       PRIMARY KEY (employee_id, phone)                                     │
│   );                                                                       │
│                                                                              │
│   -- Or array type (PostgreSQL)                                            │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       phone_numbers TEXT[]                                                 │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.3 Stored vs Derived Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 STORED vs DERIVED ATTRIBUTES                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   STORED: Physically saved in the database                                 │
│   • date_of_birth                                                          │
│   • hire_date                                                              │
│   • order_quantity                                                         │
│                                                                              │
│   DERIVED: Calculated from other attributes (may or may not be stored)    │
│   • age (from date_of_birth)                                              │
│   • tenure (from hire_date)                                               │
│   • order_total (from items * prices)                                     │
│                                                                              │
│   ER Diagram notation: Dashed ellipse for derived                         │
│                                                                              │
│   SQL Options:                                                              │
│   -- Option 1: Calculate on query                                          │
│   SELECT                                                                   │
│       date_of_birth,                                                       │
│       EXTRACT(YEAR FROM AGE(date_of_birth)) AS age                        │
│   FROM employees;                                                          │
│                                                                              │
│   -- Option 2: Generated column (computed at storage)                      │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       date_of_birth DATE,                                                  │
│       age INT GENERATED ALWAYS AS                                          │
│           (EXTRACT(YEAR FROM AGE(date_of_birth))) STORED                   │
│   );                                                                       │
│                                                                              │
│   -- Option 3: Stored for performance (denormalized)                       │
│   -- Must be kept in sync via triggers or application                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.4 Key Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       KEY ATTRIBUTES                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   KEY ATTRIBUTE: Uniquely identifies each entity instance                  │
│                                                                              │
│   Types of keys:                                                            │
│   • Natural Key: Inherent to the entity (ssn, email, isbn)                │
│   • Surrogate Key: System-generated (auto-increment, UUID)                │
│   • Composite Key: Multiple attributes together                           │
│                                                                              │
│   ER Diagram notation: Underlined attribute name                           │
│                                                                              │
│   ┌────────────────────────┐                                                │
│   │       EMPLOYEE         │                                                │
│   ├────────────────────────┤                                                │
│   │ employee_id (PK)       │  ← Key attribute (underlined)                 │
│   │ ssn (unique)           │  ← Candidate key                              │
│   │ email (unique)         │  ← Candidate key                              │
│   │ name                   │                                                │
│   └────────────────────────┘                                                │
│                                                                              │
│   Composite key example:                                                   │
│   ┌────────────────────────┐                                                │
│   │     ORDER_ITEM         │                                                │
│   ├────────────────────────┤                                                │
│   │ order_id (PK)          │                                                │
│   │ product_id (PK)        │  ← Together form primary key                  │
│   │ quantity               │                                                │
│   └────────────────────────┘                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Attribute Domain and Constraints

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  ATTRIBUTE DOMAINS AND CONSTRAINTS                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   DOMAIN: Set of valid values for an attribute                             │
│                                                                              │
│   Examples:                                                                 │
│   • age: INTEGER, 0-150                                                    │
│   • email: VARCHAR, must match email pattern                               │
│   • status: ENUM('active', 'inactive', 'pending')                         │
│   • price: DECIMAL(10,2), >= 0                                            │
│                                                                              │
│   SQL Implementation:                                                       │
│   CREATE TYPE order_status AS ENUM                                         │
│       ('pending', 'processing', 'shipped', 'delivered');                   │
│                                                                              │
│   CREATE TABLE orders (                                                     │
│       id INT PRIMARY KEY,                                                  │
│       status order_status NOT NULL DEFAULT 'pending',                      │
│       total DECIMAL(10,2) CHECK (total >= 0),                             │
│       customer_age INT CHECK (customer_age BETWEEN 18 AND 120)            │
│   );                                                                       │
│                                                                              │
│   Common constraints:                                                       │
│   • NOT NULL: Must have a value                                           │
│   • UNIQUE: Value must be unique across table                             │
│   • CHECK: Value must satisfy condition                                    │
│   • DEFAULT: Value if none provided                                       │
│   • FOREIGN KEY: References another table                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Best Practices for Entity and Attribute Design

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DESIGN BEST PRACTICES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   NAMING CONVENTIONS                                                        │
│   • Use singular nouns for entities (Customer, not Customers)              │
│   • Use snake_case or camelCase consistently                               │
│   • Be descriptive but concise                                             │
│   • Avoid reserved words                                                   │
│                                                                              │
│   ENTITY DESIGN                                                             │
│   • Each entity should represent one concept                               │
│   • Include all relevant attributes                                        │
│   • Identify strong vs weak entities                                       │
│   • Consider entity lifecycle                                              │
│                                                                              │
│   ATTRIBUTE DESIGN                                                          │
│   • Choose appropriate data types                                          │
│   • Define constraints early                                               │
│   • Avoid calculated/derived storage unless needed                         │
│   • Handle multi-valued attributes properly                                │
│                                                                              │
│   KEY SELECTION                                                             │
│   • Prefer surrogate keys for flexibility                                  │
│   • Keep natural keys as UNIQUE constraints                                │
│   • Avoid using business data as primary key                              │
│   • Consider composite keys carefully                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Summary

| Concept | Description | Example |
|---------|-------------|---------|
| Strong Entity | Independent existence | Customer, Product |
| Weak Entity | Depends on parent | OrderItem, Dependent |
| Associative Entity | Resolves M:N with attributes | Enrollment |
| Simple Attribute | Atomic, indivisible | age, id |
| Composite Attribute | Can be broken down | full_name, address |
| Multi-valued Attribute | Multiple values per entity | phone_numbers |
| Derived Attribute | Calculated from others | age (from DOB) |
| Key Attribute | Uniquely identifies | employee_id |
