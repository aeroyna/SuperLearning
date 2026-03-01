# Advanced ER Concepts

## Learning Objectives
- Understand generalization and specialization hierarchies
- Master aggregation and composition relationships
- Learn advanced constraint representations
- Apply enhanced ER modeling techniques

---

## 1. Generalization and Specialization

### Generalization (Bottom-Up)
Combining multiple entity types into a higher-level supertype based on common attributes.

```
┌─────────────────────────────────────┐
│            EMPLOYEE                 │
│  (emp_id, name, hire_date, salary)  │
└─────────────────────────────────────┘
           ▲           ▲
           │           │
    ┌──────┴──┐    ┌───┴──────┐
    │ MANAGER │    │ ENGINEER │
    │(dept_id)│    │(specialty│
    └─────────┘    │ cert_date)│
                   └──────────┘
```

### Specialization (Top-Down)
Defining subtypes from a supertype based on distinguishing characteristics.

```sql
-- Supertype table
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2),
    emp_type VARCHAR(20) NOT NULL CHECK (emp_type IN ('MANAGER', 'ENGINEER', 'SALES'))
);

-- Subtype tables
CREATE TABLE manager (
    emp_id INT PRIMARY KEY REFERENCES employee(emp_id),
    department_id INT REFERENCES department(dept_id),
    budget_authority DECIMAL(15,2)
);

CREATE TABLE engineer (
    emp_id INT PRIMARY KEY REFERENCES employee(emp_id),
    specialty VARCHAR(50),
    certification_date DATE
);
```

### Inheritance Constraints

#### Disjoint vs. Overlapping

**Disjoint (Exclusive)**: An entity can belong to only ONE subtype.

```
┌─────────────┐
│   VEHICLE   │
├─────────────┤
│ vehicle_id  │
│ make        │
│ model       │
└──────┬──────┘
       │{disjoint}
   ┌───┴───┐
   │       │
┌──▼──┐ ┌──▼──┐
│ CAR │ │TRUCK│
└─────┘ └─────┘
```

```sql
-- Enforcing disjoint constraint
CREATE TABLE vehicle (
    vehicle_id INT PRIMARY KEY,
    make VARCHAR(50),
    model VARCHAR(50),
    vehicle_type VARCHAR(10) NOT NULL CHECK (vehicle_type IN ('CAR', 'TRUCK'))
);

-- Only one subtype row per vehicle
CREATE TABLE car (
    vehicle_id INT PRIMARY KEY REFERENCES vehicle(vehicle_id),
    num_doors INT,
    CONSTRAINT chk_car_type CHECK (
        EXISTS (SELECT 1 FROM vehicle v WHERE v.vehicle_id = car.vehicle_id AND v.vehicle_type = 'CAR')
    )
);
```

**Overlapping**: An entity can belong to multiple subtypes.

```
┌─────────────┐
│   PERSON    │
├─────────────┤
│ person_id   │
│ name        │
└──────┬──────┘
       │{overlapping}
   ┌───┴───┐
   │       │
┌──▼──┐ ┌──▼───┐
│STAFF│ │STUDENT│
└─────┘ └──────┘

-- A person can be BOTH staff AND student
```

```sql
-- Overlapping: Person can be in multiple subtypes
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE staff (
    person_id INT PRIMARY KEY REFERENCES person(person_id),
    hire_date DATE,
    department VARCHAR(50)
);

CREATE TABLE student (
    person_id INT PRIMARY KEY REFERENCES person(person_id),
    enrollment_date DATE,
    program VARCHAR(50)
);

-- Same person can exist in both staff and student tables
```

#### Total vs. Partial Participation

**Total Participation**: Every supertype entity MUST be a member of at least one subtype.

```
┌─────────────┐
│   ACCOUNT   │
└──────┬──────┘
       │{total}
   ┌───┼───┐
   │   │   │
┌──▼┐┌─▼─┐┌▼──┐
│SAV││CHK││INV│
└───┘└───┘└───┘

-- Every account MUST be savings, checking, or investment
```

**Partial Participation**: Supertype entities may or may not belong to any subtype.

```
┌─────────────┐
│   EMPLOYEE  │
└──────┬──────┘
       │{partial}
   ┌───┴───┐
   │       │
┌──▼──┐ ┌──▼──┐
│PILOT│ │MECHANIC│
└─────┘ └────────┘

-- Some employees may be neither pilots nor mechanics
```

---

## 2. Aggregation

Aggregation treats a relationship as a higher-level entity that participates in another relationship.

### Problem Without Aggregation

```
PROJECT ─────────── uses ─────────── MACHINE
                      │
                      │ monitored_by
                      ▼
                  TECHNICIAN

-- How do we say "Technician monitors which PROJECT-uses-MACHINE combination"?
```

### Solution with Aggregation

```
┌─────────────────────────────────────────┐
│          PROJECT_USES_MACHINE           │
│   ┌─────────┐       ┌─────────┐         │
│   │ PROJECT │──uses─│ MACHINE │         │
│   └─────────┘       └─────────┘         │
└─────────────────────┬───────────────────┘
                      │
                      │ monitored_by
                      ▼
                ┌──────────┐
                │TECHNICIAN│
                └──────────┘
```

```sql
-- First, create the aggregated relationship
CREATE TABLE project_machine_assignment (
    assignment_id INT PRIMARY KEY,
    project_id INT REFERENCES project(project_id),
    machine_id INT REFERENCES machine(machine_id),
    start_date DATE,
    UNIQUE(project_id, machine_id)
);

-- Then, relate technician to the aggregated entity
CREATE TABLE monitoring (
    monitoring_id INT PRIMARY KEY,
    assignment_id INT REFERENCES project_machine_assignment(assignment_id),
    technician_id INT REFERENCES technician(tech_id),
    shift VARCHAR(20),
    monitoring_start DATE
);
```

### Real-World Aggregation Example: Course Enrollment

```
┌─────────────────────────────────────────┐
│              ENROLLMENT                  │
│   ┌─────────┐          ┌────────┐       │
│   │ STUDENT │──enrolls─│ COURSE │       │
│   └─────────┘          └────────┘       │
└─────────────────────┬───────────────────┘
                      │
                      │ evaluated_by
                      ▼
                ┌───────────┐
                │ PROFESSOR │
                └───────────┘
```

---

## 3. Multi-Valued and Composite Attributes

### Multi-Valued Attributes

Attributes that can hold multiple values for a single entity.

```
┌────────────────────────────────┐
│           EMPLOYEE             │
├────────────────────────────────┤
│ emp_id (PK)                    │
│ name                           │
│ {phone_numbers}  ← multi-valued│
│ {skills}         ← multi-valued│
└────────────────────────────────┘
```

```sql
-- Main entity table
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Separate table for multi-valued attribute
CREATE TABLE employee_phone (
    emp_id INT REFERENCES employee(emp_id) ON DELETE CASCADE,
    phone_number VARCHAR(20),
    phone_type VARCHAR(20) DEFAULT 'mobile',
    PRIMARY KEY (emp_id, phone_number)
);

CREATE TABLE employee_skill (
    emp_id INT REFERENCES employee(emp_id) ON DELETE CASCADE,
    skill_name VARCHAR(50),
    proficiency_level INT CHECK (proficiency_level BETWEEN 1 AND 5),
    PRIMARY KEY (emp_id, skill_name)
);
```

### Composite Attributes

Attributes that can be divided into smaller sub-parts.

```
┌────────────────────────────────────────┐
│              EMPLOYEE                   │
├────────────────────────────────────────┤
│ emp_id (PK)                            │
│ full_name                              │
│   ├── first_name                       │
│   ├── middle_name                      │
│   └── last_name                        │
│ address                                │
│   ├── street                           │
│   ├── city                             │
│   ├── state                            │
│   └── zip_code                         │
└────────────────────────────────────────┘
```

```sql
-- Option 1: Flatten into single table
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    middle_name VARCHAR(50),
    last_name VARCHAR(50),
    street VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(50),
    zip_code VARCHAR(20)
);

-- Option 2: Use structured types (PostgreSQL)
CREATE TYPE name_type AS (
    first_name VARCHAR(50),
    middle_name VARCHAR(50),
    last_name VARCHAR(50)
);

CREATE TYPE address_type AS (
    street VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(50),
    zip_code VARCHAR(20)
);

CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    full_name name_type,
    address address_type
);
```

---

## 4. Derived Attributes

Attributes whose values are calculated from other attributes.

```
┌─────────────────────────────────────┐
│             EMPLOYEE                 │
├─────────────────────────────────────┤
│ emp_id (PK)                         │
│ birth_date                          │
│ /age          ← derived             │
│ hourly_rate                         │
│ hours_worked                        │
│ /total_pay    ← derived             │
└─────────────────────────────────────┘
```

```sql
-- Implementation options for derived attributes

-- Option 1: Computed/Generated Columns (Modern SQL)
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    birth_date DATE NOT NULL,
    age INT GENERATED ALWAYS AS (
        EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date))
    ) STORED,
    hourly_rate DECIMAL(10,2),
    hours_worked DECIMAL(10,2),
    total_pay DECIMAL(10,2) GENERATED ALWAYS AS (hourly_rate * hours_worked) STORED
);

-- Option 2: Views
CREATE VIEW employee_with_derived AS
SELECT
    emp_id,
    birth_date,
    EXTRACT(YEAR FROM AGE(CURRENT_DATE, birth_date)) AS age,
    hourly_rate,
    hours_worked,
    hourly_rate * hours_worked AS total_pay
FROM employee;

-- Option 3: Virtual columns (MySQL 5.7+)
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    birth_date DATE NOT NULL,
    age INT AS (TIMESTAMPDIFF(YEAR, birth_date, CURDATE())) VIRTUAL,
    hourly_rate DECIMAL(10,2),
    hours_worked DECIMAL(10,2),
    total_pay DECIMAL(10,2) AS (hourly_rate * hours_worked) STORED
);
```

---

## 5. Ternary and N-ary Relationships

### Ternary Relationships

Relationships involving exactly three entity types.

```
        ┌──────────┐
        │ SUPPLIER │
        └────┬─────┘
             │
             ▼
┌────────┐        ┌─────────┐
│  PART  │◄──SUPPLY──►│ PROJECT │
└────────┘        └─────────┘

-- SUPPLY: Which supplier provides which part to which project
```

```sql
-- Ternary relationship table
CREATE TABLE supply (
    supplier_id INT REFERENCES supplier(supplier_id),
    part_id INT REFERENCES part(part_id),
    project_id INT REFERENCES project(project_id),
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2),
    supply_date DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (supplier_id, part_id, project_id)
);
```

### Why Not Use Binary Relationships?

```
-- Three binary relationships CANNOT capture the same semantics:

supplier_provides_part(supplier_id, part_id)
part_used_in_project(part_id, project_id)
supplier_works_with_project(supplier_id, project_id)

-- These cannot tell us: "Supplier S1 provides Part P1 specifically to Project Proj1"
-- vs "Supplier S1 provides Part P1 to Project Proj2"
```

### N-ary Relationship Example (Quaternary)

```
┌──────────┐  ┌─────────┐  ┌──────────┐  ┌──────────┐
│ STUDENT  │  │ COURSE  │  │ SEMESTER │  │INSTRUCTOR│
└────┬─────┘  └────┬────┘  └────┬─────┘  └────┬─────┘
     │             │            │              │
     └─────────────┼────────────┼──────────────┘
                   │
              ┌────▼────┐
              │ENROLLMENT│
              └─────────┘
```

```sql
CREATE TABLE enrollment (
    student_id INT REFERENCES student(student_id),
    course_id INT REFERENCES course(course_id),
    semester_id INT REFERENCES semester(semester_id),
    instructor_id INT REFERENCES instructor(instructor_id),
    grade CHAR(2),
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id, semester_id, instructor_id)
);
```

---

## 6. Role Names in Relationships

When the same entity type participates multiple times in a relationship.

### Self-Referencing (Recursive) Relationships

```
┌──────────────────────────┐
│        EMPLOYEE          │
└──────────────────────────┘
    │                 ▲
    │ manages        │
    │ (manager)      │
    └────────────────┘
      (subordinate)
```

```sql
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT REFERENCES employee(emp_id),  -- Role: manager
    -- The referencing column plays role: subordinate
    CONSTRAINT no_self_manage CHECK (emp_id != manager_id)
);

-- Query using role names
SELECT
    e.name AS subordinate_name,
    m.name AS manager_name
FROM employee e
LEFT JOIN employee m ON e.manager_id = m.emp_id;
```

### Multiple Roles in Same Relationship

```
┌────────────┐                          ┌────────────┐
│   PERSON   │◄─── flight_booking ────►│   FLIGHT   │
└────────────┘                          └────────────┘
  │   │
  │   └── (passenger)
  └────── (booked_by)
```

```sql
CREATE TABLE flight_booking (
    booking_id INT PRIMARY KEY,
    passenger_id INT NOT NULL REFERENCES person(person_id),  -- Role: passenger
    booked_by_id INT NOT NULL REFERENCES person(person_id),  -- Role: booked_by
    flight_id INT NOT NULL REFERENCES flight(flight_id),
    booking_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Passenger and booked_by can be different people (e.g., parent books for child)
```

---

## 7. Enhanced ER (EER) Diagram Notation

### Complete EER Example

```
                    ┌───────────────┐
                    │    PERSON     │
                    ├───────────────┤
                    │ person_id PK  │
                    │ name          │
                    │ email         │
                    └───────┬───────┘
                            │
                     {total, disjoint}
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
    ┌─────▼─────┐    ┌─────▼─────┐    ┌─────▼─────┐
    │ CUSTOMER  │    │  EMPLOYEE │    │  VENDOR   │
    ├───────────┤    ├───────────┤    ├───────────┤
    │credit_limit│    │hire_date  │    │company    │
    │membership │    │salary     │    │tax_id     │
    └───────────┘    └─────┬─────┘    └───────────┘
                           │
                    {partial, overlapping}
                           │
               ┌───────────┼───────────┐
               │           │           │
        ┌──────▼──┐ ┌──────▼──┐ ┌──────▼──┐
        │ MANAGER │ │DEVELOPER│ │ ANALYST │
        ├─────────┤ ├─────────┤ ├─────────┤
        │budget   │ │languages│ │domain   │
        └─────────┘ └─────────┘ └─────────┘
```

### Standard EER Notations

| Symbol | Meaning |
|--------|---------|
| d | Disjoint specialization |
| o | Overlapping specialization |
| Single line | Partial participation |
| Double line | Total participation |
| (U) | Union type / Category |

---

## 8. Categories (Union Types)

A category is a subtype that has multiple supertype entities.

```
         ┌──────────┐  ┌──────────┐  ┌──────────┐
         │  PERSON  │  │ COMPANY  │  │  BANK    │
         └────┬─────┘  └────┬─────┘  └────┬─────┘
              │             │             │
              └──────┬──────┴──────┬──────┘
                     │             │
                     ▼   (U)       │
              ┌──────────────┐     │
              │    OWNER     │◄────┘
              └──────┬───────┘
                     │
                     │ owns
                     ▼
              ┌──────────────┐
              │   VEHICLE    │
              └──────────────┘

-- OWNER can be a Person, Company, OR Bank
```

```sql
-- Implementation using discriminator
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    ssn VARCHAR(11)
);

CREATE TABLE company (
    company_id INT PRIMARY KEY,
    name VARCHAR(100),
    tax_id VARCHAR(20)
);

CREATE TABLE bank (
    bank_id INT PRIMARY KEY,
    name VARCHAR(100),
    routing_number VARCHAR(20)
);

-- Category/Union table
CREATE TABLE owner (
    owner_id INT PRIMARY KEY,
    owner_type VARCHAR(10) NOT NULL CHECK (owner_type IN ('PERSON', 'COMPANY', 'BANK')),
    person_id INT REFERENCES person(person_id),
    company_id INT REFERENCES company(company_id),
    bank_id INT REFERENCES bank(bank_id),
    -- Exactly one foreign key should be non-null based on type
    CONSTRAINT valid_owner CHECK (
        (owner_type = 'PERSON' AND person_id IS NOT NULL AND company_id IS NULL AND bank_id IS NULL) OR
        (owner_type = 'COMPANY' AND company_id IS NOT NULL AND person_id IS NULL AND bank_id IS NULL) OR
        (owner_type = 'BANK' AND bank_id IS NOT NULL AND person_id IS NULL AND company_id IS NULL)
    )
);

CREATE TABLE vehicle (
    vehicle_id INT PRIMARY KEY,
    vin VARCHAR(17) UNIQUE,
    owner_id INT REFERENCES owner(owner_id)
);
```

---

## 9. Mapping EER to Relational Schema

### Strategy Options for Specialization/Generalization

#### Option 1: Single Table with NULL (Table-Per-Hierarchy)

```sql
-- All subtypes in one table
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    person_type VARCHAR(20) NOT NULL,
    -- Customer attributes (NULL for non-customers)
    credit_limit DECIMAL(10,2),
    membership_level VARCHAR(20),
    -- Employee attributes (NULL for non-employees)
    hire_date DATE,
    salary DECIMAL(10,2)
);

-- Pros: Simple queries, no joins
-- Cons: Many NULL values, storage waste, constraint enforcement complex
```

#### Option 2: Separate Tables (Table-Per-Type)

```sql
-- Supertype table
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    person_type VARCHAR(20) NOT NULL
);

-- Subtype tables
CREATE TABLE customer (
    person_id INT PRIMARY KEY REFERENCES person(person_id),
    credit_limit DECIMAL(10,2),
    membership_level VARCHAR(20)
);

CREATE TABLE employee (
    person_id INT PRIMARY KEY REFERENCES person(person_id),
    hire_date DATE,
    salary DECIMAL(10,2)
);

-- Pros: No NULL waste, clear structure
-- Cons: Requires JOINs to get complete entity
```

#### Option 3: Concrete Tables Only (Table-Per-Concrete-Class)

```sql
-- No supertype table, each subtype is complete
CREATE TABLE customer (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    credit_limit DECIMAL(10,2),
    membership_level VARCHAR(20)
);

CREATE TABLE employee (
    person_id INT PRIMARY KEY,
    name VARCHAR(100),
    hire_date DATE,
    salary DECIMAL(10,2)
);

-- Pros: Fast queries for specific types
-- Cons: Redundant columns, hard to query "all persons"
```

---

## 10. Practical EER Modeling Guidelines

### When to Use Specialization
1. Subtypes have distinct attributes
2. Subtypes have distinct relationships
3. Business rules differ by subtype
4. Application frequently works with specific subtypes

### When to Avoid Deep Hierarchies
```
-- Avoid this:
Animal → Mammal → Carnivore → Feline → Domestic Cat → Persian Cat

-- Prefer flatter designs with attributes:
Animal (animal_type, diet_type, domestication_status, breed)
```

### Practical Example: E-Commerce User Model

```sql
-- Using Option 2 (Table-Per-Type) with role separation
CREATE TABLE user_account (
    user_id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT true
);

-- Users can have multiple roles (overlapping)
CREATE TABLE customer_profile (
    user_id INT PRIMARY KEY REFERENCES user_account(user_id),
    shipping_address_id INT REFERENCES address(address_id),
    billing_address_id INT REFERENCES address(address_id),
    loyalty_points INT DEFAULT 0
);

CREATE TABLE seller_profile (
    user_id INT PRIMARY KEY REFERENCES user_account(user_id),
    store_name VARCHAR(100),
    commission_rate DECIMAL(5,2),
    verified_at TIMESTAMP
);

CREATE TABLE admin_profile (
    user_id INT PRIMARY KEY REFERENCES user_account(user_id),
    access_level INT NOT NULL,
    last_admin_action TIMESTAMP
);

-- A user can be customer AND seller (overlapping allowed)
```

---

## Summary

| Concept | Description | Key Consideration |
|---------|-------------|-------------------|
| Generalization | Combine entities into supertype | Bottom-up design |
| Specialization | Define subtypes from supertype | Top-down design |
| Disjoint | Entity in only one subtype | Use discriminator column |
| Overlapping | Entity in multiple subtypes | Separate subtype tables |
| Total | Must be in at least one subtype | Enforce via triggers/constraints |
| Partial | May not be in any subtype | Allow NULL discriminator |
| Aggregation | Relationship as entity | Create intermediate table |
| Category/Union | Multiple possible supertypes | Use discriminator + multiple FKs |

---

## Practice Exercises

1. Design an EER diagram for a university system with Person generalized into Student, Faculty, and Staff (some can overlap)

2. Model a vehicle hierarchy: Vehicle → {Automobile, Motorcycle, Truck} with disjoint constraint, where Automobile further specializes into {Sedan, SUV, Coupe}

3. Create an aggregation for: "Mechanic services which Car at which ServiceCenter"

4. Implement a category where INSURANCE_POLICY can cover either a PERSON, VEHICLE, or PROPERTY

---

## Further Reading

- **"Fundamentals of Database Systems"** - Elmasri & Navathe, Chapter 4
- **"Database System Concepts"** - Silberschatz et al., EER chapter
- **UML to Database Mapping** - Comparing EER with UML class diagrams
