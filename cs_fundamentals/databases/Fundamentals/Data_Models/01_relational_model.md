# Relational Model

## 1. Introduction

The **relational model** was introduced by Edgar F. Codd in 1970 and remains the most widely used data model. It organizes data into **relations** (tables) with **tuples** (rows) and **attributes** (columns).

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RELATIONAL MODEL STRUCTURE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   RELATION (Table): employees                                                │
│   ┌────────────┬────────────┬─────────────┬──────────┬──────────────┐       │
│   │ ATTRIBUTE  │ ATTRIBUTE  │  ATTRIBUTE  │ATTRIBUTE │  ATTRIBUTE   │       │
│   │   (id)     │  (name)    │   (email)   │ (dept_id)│   (salary)   │       │
│   ├────────────┼────────────┼─────────────┼──────────┼──────────────┤       │
│   │     1      │   Alice    │ alice@x.com │    10    │   75000      │ TUPLE │
│   │     2      │    Bob     │  bob@x.com  │    20    │   82000      │ (Row) │
│   │     3      │  Charlie   │charlie@x.com│    10    │   65000      │       │
│   └────────────┴────────────┴─────────────┴──────────┴──────────────┘       │
│                                                                              │
│   DOMAIN: Set of allowed values for each attribute                          │
│   - id: INTEGER                                                              │
│   - name: VARCHAR(100)                                                       │
│   - email: VARCHAR(255)                                                      │
│   - dept_id: INTEGER (Foreign Key)                                          │
│   - salary: DECIMAL(10,2)                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Concepts

### 2.1 Relations (Tables)

A relation is a named, two-dimensional table with the following properties:

```sql
CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    department_id INTEGER,
    salary DECIMAL(10, 2),
    hire_date DATE DEFAULT CURRENT_DATE,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

**Properties of Relations:**
1. **No duplicate tuples** (enforced by primary key)
2. **Order of tuples is insignificant**
3. **Order of attributes is insignificant**
4. **All attribute values are atomic** (no nested structures)
5. **Each attribute has a distinct name**

### 2.2 Keys

```sql
-- Primary Key: Uniquely identifies each tuple
CREATE TABLE products (
    product_id INTEGER PRIMARY KEY,  -- Simple primary key
    name VARCHAR(200)
);

-- Composite Primary Key: Multiple columns
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)  -- Composite key
);

-- Foreign Key: References another table
CREATE TABLE orders (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Candidate Key: Could be primary key
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY,
    email VARCHAR(255) UNIQUE,      -- Candidate key
    username VARCHAR(50) UNIQUE     -- Candidate key
);
```

**Key Types:**

| Key Type | Description |
|----------|-------------|
| **Super Key** | Any set of attributes that uniquely identifies tuples |
| **Candidate Key** | Minimal super key (no proper subset is a super key) |
| **Primary Key** | Chosen candidate key for identification |
| **Foreign Key** | References primary key in another table |
| **Alternate Key** | Candidate keys not chosen as primary |

### 2.3 Constraints

```sql
CREATE TABLE employees (
    id INTEGER PRIMARY KEY,                              -- Entity integrity
    name VARCHAR(100) NOT NULL,                          -- NOT NULL constraint
    email VARCHAR(255) UNIQUE,                           -- Uniqueness constraint
    age INTEGER CHECK (age >= 18 AND age <= 120),        -- Check constraint
    salary DECIMAL(10,2) CHECK (salary > 0),
    department_id INTEGER REFERENCES departments(id),     -- Referential integrity
    status VARCHAR(20) DEFAULT 'active'                  -- Default constraint
);

-- Table-level constraints
CREATE TABLE project_assignments (
    employee_id INTEGER,
    project_id INTEGER,
    role VARCHAR(50),
    start_date DATE,
    end_date DATE,
    PRIMARY KEY (employee_id, project_id),
    CHECK (end_date > start_date OR end_date IS NULL),
    FOREIGN KEY (employee_id) REFERENCES employees(id),
    FOREIGN KEY (project_id) REFERENCES projects(id)
);
```

---

## 3. Relational Algebra

The mathematical foundation for SQL operations.

### 3.1 Selection (σ)

Filters rows based on a condition.

```
σ_condition(Relation)

σ_{salary > 50000}(employees)
```

```sql
-- SQL equivalent
SELECT * FROM employees WHERE salary > 50000;
```

### 3.2 Projection (π)

Selects specific columns.

```
π_{attribute1, attribute2}(Relation)

π_{name, email}(employees)
```

```sql
-- SQL equivalent
SELECT name, email FROM employees;
```

### 3.3 Union (∪)

Combines tuples from two relations.

```
Relation1 ∪ Relation2
```

```sql
-- SQL equivalent
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```

### 3.4 Intersection (∩)

Tuples present in both relations.

```sql
-- SQL equivalent
SELECT name FROM employees
INTERSECT
SELECT name FROM managers;
```

### 3.5 Difference (−)

Tuples in first but not in second relation.

```sql
-- SQL equivalent
SELECT name FROM employees
EXCEPT
SELECT name FROM retired_employees;
```

### 3.6 Cartesian Product (×)

All combinations of tuples from two relations.

```sql
-- SQL equivalent (rarely used directly)
SELECT * FROM employees, departments;
-- or
SELECT * FROM employees CROSS JOIN departments;
```

### 3.7 Join (⋈)

Combines related tuples from two relations.

```
┌───────────────────────────────────────────────────────────────────────────┐
│                           JOIN TYPES                                       │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│   Table A            Table B              Result                           │
│   ┌───┬───┐         ┌───┬───┐                                             │
│   │ 1 │ a │         │ 1 │ x │                                             │
│   │ 2 │ b │         │ 2 │ y │                                             │
│   │ 3 │ c │         │ 4 │ z │                                             │
│   └───┴───┘         └───┴───┘                                             │
│                                                                            │
│   INNER JOIN:       Only matching rows                                     │
│   ┌───┬───┬───┐                                                           │
│   │ 1 │ a │ x │                                                           │
│   │ 2 │ b │ y │                                                           │
│   └───┴───┴───┘                                                           │
│                                                                            │
│   LEFT JOIN:        All from A, matching from B                           │
│   ┌───┬───┬──────┐                                                        │
│   │ 1 │ a │  x   │                                                        │
│   │ 2 │ b │  y   │                                                        │
│   │ 3 │ c │ NULL │                                                        │
│   └───┴───┴──────┘                                                        │
│                                                                            │
│   RIGHT JOIN:       All from B, matching from A                           │
│   ┌──────┬──────┬───┐                                                     │
│   │  1   │  a   │ x │                                                     │
│   │  2   │  b   │ y │                                                     │
│   │ NULL │ NULL │ z │                                                     │
│   └──────┴──────┴───┘                                                     │
│                                                                            │
│   FULL OUTER JOIN:  All from both                                         │
│   ┌──────┬──────┬──────┐                                                  │
│   │  1   │  a   │  x   │                                                  │
│   │  2   │  b   │  y   │                                                  │
│   │  3   │  c   │ NULL │                                                  │
│   │ NULL │ NULL │  z   │                                                  │
│   └──────┴──────┴──────┘                                                  │
│                                                                            │
└───────────────────────────────────────────────────────────────────────────┘
```

```sql
-- INNER JOIN
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- LEFT JOIN
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- RIGHT JOIN
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;

-- FULL OUTER JOIN
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- SELF JOIN
SELECT e1.name as employee, e2.name as manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id;
```

---

## 4. Schema Design

### 4.1 One-to-One Relationship

```sql
-- User and UserProfile (1:1)
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE user_profiles (
    user_id INTEGER PRIMARY KEY,  -- Same as users.id
    bio TEXT,
    avatar_url VARCHAR(500),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### 4.2 One-to-Many Relationship

```sql
-- Department has many Employees (1:N)
CREATE TABLE departments (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INTEGER,  -- Foreign key on "many" side
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

### 4.3 Many-to-Many Relationship

```sql
-- Students and Courses (M:N) via Junction Table
CREATE TABLE students (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE courses (
    id INTEGER PRIMARY KEY,
    title VARCHAR(200) NOT NULL
);

-- Junction/Association table
CREATE TABLE enrollments (
    student_id INTEGER,
    course_id INTEGER,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);

-- Query: Get all courses for a student
SELECT c.title, e.grade
FROM courses c
JOIN enrollments e ON c.id = e.course_id
WHERE e.student_id = 1;

-- Query: Get all students in a course
SELECT s.name, e.grade
FROM students s
JOIN enrollments e ON s.id = e.student_id
WHERE e.course_id = 101;
```

---

## 5. Advanced SQL Concepts

### 5.1 Subqueries

```sql
-- Scalar subquery
SELECT name, salary,
       (SELECT AVG(salary) FROM employees) as avg_salary
FROM employees;

-- Subquery in WHERE
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM (derived table)
SELECT dept_name, avg_salary
FROM (
    SELECT d.name as dept_name, AVG(e.salary) as avg_salary
    FROM departments d
    JOIN employees e ON d.id = e.department_id
    GROUP BY d.name
) dept_stats
WHERE avg_salary > 60000;

-- EXISTS subquery
SELECT d.name
FROM departments d
WHERE EXISTS (
    SELECT 1 FROM employees e
    WHERE e.department_id = d.id AND e.salary > 100000
);

-- IN subquery
SELECT name FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE location = 'New York'
);
```

### 5.2 Common Table Expressions (CTEs)

```sql
-- Simple CTE
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 80000
)
SELECT d.name, COUNT(*) as high_earner_count
FROM departments d
JOIN high_earners h ON d.id = h.department_id
GROUP BY d.name;

-- Recursive CTE (organizational hierarchy)
WITH RECURSIVE org_hierarchy AS (
    -- Base case: top-level managers
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: employees under managers
    SELECT e.id, e.name, e.manager_id, h.level + 1
    FROM employees e
    JOIN org_hierarchy h ON e.manager_id = h.id
)
SELECT * FROM org_hierarchy ORDER BY level, name;
```

### 5.3 Window Functions

```sql
-- Row number, rank, and dense rank
SELECT
    name,
    department_id,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as row_num,
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as dense_rank
FROM employees;

-- Running totals and moving averages
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) as running_total,
    AVG(amount) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as moving_avg_7day
FROM orders;

-- Lead and Lag
SELECT
    name,
    salary,
    LAG(salary) OVER (ORDER BY salary) as prev_salary,
    LEAD(salary) OVER (ORDER BY salary) as next_salary
FROM employees;

-- First and Last values
SELECT
    department_id,
    name,
    salary,
    FIRST_VALUE(name) OVER (PARTITION BY department_id ORDER BY salary DESC) as highest_paid,
    LAST_VALUE(name) OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
        RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) as lowest_paid
FROM employees;
```

---

## 6. Code Examples Across Languages

### Python (SQLAlchemy ORM)

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker

Base = declarative_base()

class Department(Base):
    __tablename__ = 'departments'

    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    employees = relationship('Employee', back_populates='department')

class Employee(Base):
    __tablename__ = 'employees'

    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True)
    department_id = Column(Integer, ForeignKey('departments.id'))
    department = relationship('Department', back_populates='employees')

# Create engine and session
engine = create_engine('postgresql://user:pass@localhost/db')
Session = sessionmaker(bind=engine)
session = Session()

# Create tables
Base.metadata.create_all(engine)

# Insert data
dept = Department(name='Engineering')
emp = Employee(name='Alice', email='alice@example.com', department=dept)
session.add_all([dept, emp])
session.commit()

# Query with join
employees = session.query(Employee).join(Department).filter(
    Department.name == 'Engineering'
).all()

for emp in employees:
    print(f"{emp.name} - {emp.department.name}")
```

### Java (JPA/Hibernate)

```java
import javax.persistence.*;
import java.util.List;

@Entity
@Table(name = "departments")
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees;

    // Getters and setters
}

@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(unique = true)
    private String email;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    // Getters and setters
}

// Repository usage
public class EmployeeRepository {
    private EntityManager em;

    public List<Employee> findByDepartment(String deptName) {
        String jpql = """
            SELECT e FROM Employee e
            JOIN e.department d
            WHERE d.name = :deptName
        """;

        return em.createQuery(jpql, Employee.class)
                 .setParameter("deptName", deptName)
                 .getResultList();
    }
}
```

### JavaScript (Prisma ORM)

```javascript
// schema.prisma
/*
model Department {
  id        Int        @id @default(autoincrement())
  name      String
  employees Employee[]
}

model Employee {
  id           Int         @id @default(autoincrement())
  name         String
  email        String      @unique
  departmentId Int?
  department   Department? @relation(fields: [departmentId], references: [id])
}
*/

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
    // Create department with employees
    const department = await prisma.department.create({
        data: {
            name: 'Engineering',
            employees: {
                create: [
                    { name: 'Alice', email: 'alice@example.com' },
                    { name: 'Bob', email: 'bob@example.com' }
                ]
            }
        },
        include: { employees: true }
    });

    // Query with relations
    const engineers = await prisma.employee.findMany({
        where: {
            department: { name: 'Engineering' }
        },
        include: { department: true }
    });

    // Aggregation
    const stats = await prisma.employee.groupBy({
        by: ['departmentId'],
        _count: { id: true },
        _avg: { salary: true }
    });
}

main().finally(() => prisma.$disconnect());
```

---

## 7. Advantages and Limitations

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Data Integrity** | Constraints ensure data quality |
| **ACID Transactions** | Reliable multi-statement operations |
| **Mature Ecosystem** | Decades of tooling and expertise |
| **Powerful Queries** | SQL handles complex operations |
| **Standardization** | SQL is portable across databases |
| **Normalization** | Reduces data redundancy |

### Limitations

| Limitation | Description |
|------------|-------------|
| **Scaling** | Horizontal scaling is challenging |
| **Schema Rigidity** | Changes require migrations |
| **Object-Relational Mismatch** | Mapping objects to tables is complex |
| **Hierarchical Data** | Trees/graphs require workarounds |
| **High Volume Writes** | Can be slower than NoSQL alternatives |

---

## 8. Summary

The relational model provides:
- **Mathematical foundation** through relational algebra
- **Data integrity** through constraints and normalization
- **Powerful querying** through SQL
- **Transaction support** through ACID properties

It remains the best choice for applications requiring:
- Complex queries and reporting
- Strong consistency guarantees
- Structured data with clear relationships
- Regulatory compliance and auditability
