# Relationships and Cardinality

## 1. Introduction

**Relationships** describe how entities are associated with each other. **Cardinality** specifies how many instances of one entity relate to instances of another.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  RELATIONSHIPS AND CARDINALITY                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   RELATIONSHIP: An association between entities                             │
│   • "Customer PLACES Order"                                                │
│   • "Employee WORKS_IN Department"                                         │
│   • "Student ENROLLS_IN Course"                                            │
│                                                                              │
│   CARDINALITY: How many of each entity participate                         │
│   • One-to-One (1:1)                                                       │
│   • One-to-Many (1:N)                                                      │
│   • Many-to-Many (M:N)                                                     │
│                                                                              │
│   ┌──────────┐    places     ┌───────────┐                                 │
│   │ CUSTOMER │──────────────►│   ORDER   │                                 │
│   └──────────┘      1:N      └───────────┘                                 │
│                                                                              │
│   One customer can place many orders                                       │
│   Each order belongs to one customer                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Cardinality Types

### 2.1 One-to-One (1:1)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ONE-TO-ONE (1:1)                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Each instance of Entity A relates to exactly one instance of Entity B   │
│   and vice versa.                                                          │
│                                                                              │
│   ┌──────────┐      has       ┌──────────────┐                             │
│   │ EMPLOYEE │───────────────►│   PARKING    │                             │
│   │          │◄───────────────│    SPOT      │                             │
│   └──────────┘      1:1       └──────────────┘                             │
│                                                                              │
│   Examples:                                                                 │
│   • Employee ─ Employee_Details (vertical partitioning)                    │
│   • Country ─ Capital                                                      │
│   • Person ─ Passport                                                      │
│   • User ─ User_Profile                                                    │
│                                                                              │
│   SQL Implementation (two options):                                        │
│                                                                              │
│   Option 1: Foreign key in either table                                    │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│   CREATE TABLE parking_spots (                                             │
│       id INT PRIMARY KEY,                                                  │
│       spot_number VARCHAR(10),                                             │
│       employee_id INT UNIQUE REFERENCES employees(id)                      │
│   );                                                                       │
│                                                                              │
│   Option 2: Same primary key                                               │
│   CREATE TABLE employee_details (                                          │
│       employee_id INT PRIMARY KEY REFERENCES employees(id),               │
│       emergency_contact VARCHAR(100),                                      │
│       medical_info TEXT                                                    │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 One-to-Many (1:N)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ONE-TO-MANY (1:N)                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   One instance of Entity A relates to many instances of Entity B           │
│   Each instance of Entity B relates to one instance of Entity A            │
│                                                                              │
│   ┌────────────┐   contains    ┌───────────┐                               │
│   │ DEPARTMENT │──────────────►│ EMPLOYEE  │                               │
│   │            │◄ ─ ─ ─ ─ ─ ─ ─│           │                               │
│   └────────────┘      1:N      └───────────┘                               │
│         1                           N                                       │
│                                                                              │
│   Examples:                                                                 │
│   • Customer ─ Orders (one customer, many orders)                         │
│   • Author ─ Books (one author, many books - simplified)                  │
│   • Category ─ Products                                                    │
│   • Parent ─ Children                                                      │
│                                                                              │
│   SQL Implementation:                                                       │
│   -- Foreign key on the "many" side                                        │
│   CREATE TABLE departments (                                                │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       department_id INT REFERENCES departments(id)  -- FK here            │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Many-to-Many (M:N)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MANY-TO-MANY (M:N)                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Each instance of Entity A can relate to many of Entity B                 │
│   Each instance of Entity B can relate to many of Entity A                 │
│                                                                              │
│   ┌───────────┐    enrolls     ┌───────────┐                               │
│   │  STUDENT  │───────────────►│  COURSE   │                               │
│   │           │◄───────────────│           │                               │
│   └───────────┘      M:N       └───────────┘                               │
│         M                           N                                       │
│                                                                              │
│   Examples:                                                                 │
│   • Students ─ Courses (many students, many courses)                      │
│   • Authors ─ Books (multiple authors per book)                           │
│   • Products ─ Orders (one order can have many products)                  │
│   • Users ─ Roles                                                          │
│                                                                              │
│   SQL Implementation: Junction/Bridge table required                       │
│                                                                              │
│   CREATE TABLE students (                                                   │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100)                                                    │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE courses (                                                    │
│       id INT PRIMARY KEY,                                                  │
│       title VARCHAR(200)                                                   │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE enrollments (  -- Junction table                            │
│       student_id INT REFERENCES students(id),                              │
│       course_id INT REFERENCES courses(id),                                │
│       enrollment_date DATE,                                                │
│       grade VARCHAR(2),                                                    │
│       PRIMARY KEY (student_id, course_id)                                  │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Participation Constraints

### 3.1 Total vs Partial Participation

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PARTICIPATION CONSTRAINTS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   TOTAL PARTICIPATION (mandatory):                                          │
│   Every entity instance MUST participate in the relationship               │
│   Shown with double lines in ER diagrams                                   │
│                                                                              │
│   PARTIAL PARTICIPATION (optional):                                         │
│   Entity instances MAY participate in the relationship                     │
│   Shown with single lines in ER diagrams                                   │
│                                                                              │
│   ┌────────────┐              ┌───────────┐                                │
│   │ DEPARTMENT │══════════════│ EMPLOYEE  │                                │
│   └────────────┘    works_in  └───────────┘                                │
│        ║                           │                                        │
│        ║                           │                                        │
│     Total                       Partial                                     │
│   (every dept                 (not every                                   │
│    has employees)              employee in dept?)                          │
│                                                                              │
│   Real meaning:                                                             │
│   • Every department MUST have at least one employee (total)              │
│   • An employee MAY not be assigned to a department (partial)             │
│                                                                              │
│   SQL Implementation:                                                       │
│   -- Total: Use NOT NULL on foreign key                                    │
│   -- Can't easily enforce "at least one" without triggers                  │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       department_id INT NOT NULL REFERENCES departments(id)  -- Total     │
│   );                                                                       │
│                                                                              │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       department_id INT REFERENCES departments(id)  -- Partial (NULL ok)  │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Cardinality Notation Systems

### 4.1 Common Notations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CARDINALITY NOTATIONS                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   CHEN NOTATION (classic ER):                                              │
│   1 ─────── N                                                               │
│   1 ─────── 1                                                               │
│   M ─────── N                                                               │
│                                                                              │
│   CROW'S FOOT NOTATION (most common):                                      │
│   ────┤├───    One and only one                                            │
│   ───○┤├───    Zero or one                                                 │
│   ────┤<───    One or many                                                 │
│   ───○<───     Zero or many                                                │
│                                                                              │
│   Examples in Crow's Foot:                                                 │
│                                                                              │
│   One-to-Many (required):                                                  │
│   ┌──────────┐           ┌───────────┐                                     │
│   │ Customer │───────┤├──┤< Order    │                                     │
│   └──────────┘           └───────────┘                                     │
│   One customer → many orders, each order has one customer                  │
│                                                                              │
│   Many-to-Many (optional):                                                  │
│   ┌──────────┐           ┌───────────┐                                     │
│   │ Student  │○───────<──○< Course   │                                     │
│   └──────────┘           └───────────┘                                     │
│   Students can have 0+ courses, courses can have 0+ students              │
│                                                                              │
│   MIN-MAX NOTATION:                                                         │
│   (0,1) ─────── (1,N)                                                      │
│   (minimum, maximum) on each side                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Relationship Types

### 5.1 Binary Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BINARY RELATIONSHIP                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Involves exactly TWO entities (most common)                              │
│                                                                              │
│   ┌──────────┐    purchases    ┌───────────┐                               │
│   │ CUSTOMER │─────────────────│  PRODUCT  │                               │
│   └──────────┘                 └───────────┘                               │
│                                                                              │
│   ┌──────────┐    works_for    ┌───────────┐                               │
│   │ EMPLOYEE │─────────────────│  COMPANY  │                               │
│   └──────────┘                 └───────────┘                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Ternary Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TERNARY RELATIONSHIP                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Involves exactly THREE entities                                          │
│                                                                              │
│              ┌───────────┐                                                  │
│              │  PROJECT  │                                                  │
│              └─────┬─────┘                                                  │
│                    │                                                        │
│                    │                                                        │
│            ┌───────┴───────┐                                               │
│            │   SUPPLIES    │                                               │
│            └───────────────┘                                               │
│           ╱                 ╲                                              │
│          ╱                   ╲                                             │
│   ┌──────────┐         ┌──────────┐                                        │
│   │ SUPPLIER │         │   PART   │                                        │
│   └──────────┘         └──────────┘                                        │
│                                                                              │
│   "Supplier S supplies Part P to Project J"                                │
│   All three are needed to describe the relationship                        │
│                                                                              │
│   SQL Implementation:                                                       │
│   CREATE TABLE supplies (                                                   │
│       supplier_id INT REFERENCES suppliers(id),                            │
│       part_id INT REFERENCES parts(id),                                    │
│       project_id INT REFERENCES projects(id),                              │
│       quantity INT,                                                        │
│       PRIMARY KEY (supplier_id, part_id, project_id)                      │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Recursive (Unary) Relationships

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  RECURSIVE RELATIONSHIP                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Entity relates to itself                                                 │
│                                                                              │
│   Example: Employee manages Employee                                        │
│                                                                              │
│          ┌────────────────┐                                                 │
│          │    EMPLOYEE    │                                                 │
│          └───────┬────────┘                                                 │
│                  │                                                          │
│          manages │                                                          │
│                  │                                                          │
│          ┌───────┴────────┐                                                 │
│          │ manager  │ subordinate                                          │
│          │   (1)    │    (N)                                               │
│          └──────────┘                                                       │
│                                                                              │
│   Other examples:                                                           │
│   • Person is_friend_of Person                                            │
│   • Course is_prerequisite_of Course                                      │
│   • Category is_subcategory_of Category                                   │
│                                                                              │
│   SQL Implementation:                                                       │
│   CREATE TABLE employees (                                                  │
│       id INT PRIMARY KEY,                                                  │
│       name VARCHAR(100),                                                   │
│       manager_id INT REFERENCES employees(id)  -- Self-reference          │
│   );                                                                       │
│                                                                              │
│   -- Many-to-many self-reference                                           │
│   CREATE TABLE friendships (                                               │
│       person1_id INT REFERENCES persons(id),                               │
│       person2_id INT REFERENCES persons(id),                               │
│       since DATE,                                                          │
│       PRIMARY KEY (person1_id, person2_id)                                │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Relationship Attributes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   RELATIONSHIP ATTRIBUTES                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Relationships can have their own attributes                              │
│                                                                              │
│   ┌──────────┐    enrolls     ┌───────────┐                                │
│   │ STUDENT  │────────────────│  COURSE   │                                │
│   └──────────┘       │        └───────────┘                                │
│                      │                                                      │
│               ┌──────┴──────┐                                              │
│               │ grade       │                                              │
│               │ semester    │                                              │
│               │ enroll_date │                                              │
│               └─────────────┘                                              │
│                                                                              │
│   grade, semester, enroll_date belong to the RELATIONSHIP,                │
│   not to Student or Course alone                                           │
│                                                                              │
│   For M:N relationships → attributes go in junction table                 │
│   For 1:N relationships → attributes can go on the "many" side            │
│                                                                              │
│   SQL:                                                                      │
│   CREATE TABLE enrollments (                                               │
│       student_id INT REFERENCES students(id),                              │
│       course_id INT REFERENCES courses(id),                                │
│       grade VARCHAR(2),           -- Relationship attribute               │
│       semester VARCHAR(20),       -- Relationship attribute               │
│       enrollment_date DATE,       -- Relationship attribute               │
│       PRIMARY KEY (student_id, course_id)                                  │
│   );                                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Converting Relationships to Tables

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                CONVERTING RELATIONSHIPS TO SQL                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1:1 RELATIONSHIP                                                          │
│   • Add FK to either table (prefer the one with total participation)      │
│   • Or use same PK for both tables                                        │
│                                                                              │
│   1:N RELATIONSHIP                                                          │
│   • Add FK to the "many" side table                                       │
│   • Relationship attributes go with FK                                     │
│                                                                              │
│   M:N RELATIONSHIP                                                          │
│   • Create junction table with FKs to both entities                       │
│   • PK is combination of both FKs (or add surrogate key)                  │
│   • Relationship attributes go in junction table                          │
│                                                                              │
│   TERNARY RELATIONSHIP                                                      │
│   • Create junction table with FKs to all three entities                  │
│   • PK includes all three FKs (or subset if semantically appropriate)    │
│                                                                              │
│   RECURSIVE RELATIONSHIP                                                    │
│   • 1:N: Self-referencing FK in same table                                │
│   • M:N: Separate junction table referencing same table twice             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Summary

| Cardinality | Description | SQL Implementation |
|-------------|-------------|-------------------|
| 1:1 | One-to-One | FK with UNIQUE, or shared PK |
| 1:N | One-to-Many | FK on "many" side |
| M:N | Many-to-Many | Junction table with two FKs |

| Participation | Meaning | SQL Enforcement |
|---------------|---------|-----------------|
| Total | Must participate | NOT NULL on FK |
| Partial | May participate | FK allows NULL |

| Relationship Type | Description | Example |
|-------------------|-------------|---------|
| Binary | Two entities | Customer-Order |
| Ternary | Three entities | Supplier-Part-Project |
| Recursive | Self-referencing | Employee manages Employee |
