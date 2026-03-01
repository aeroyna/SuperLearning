# Second Normal Form (2NF)

## 1. Introduction

**Second Normal Form (2NF)** builds on 1NF by eliminating **partial dependencies**. It ensures that all non-key attributes depend on the entire primary key, not just part of it.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        2NF REQUIREMENTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A table is in 2NF if:                                                     │
│                                                                              │
│   1. It is in 1NF                                                           │
│   2. Every non-key attribute is FULLY dependent on the ENTIRE primary key  │
│      (no partial dependencies)                                              │
│                                                                              │
│   Note: 2NF only applies to tables with COMPOSITE primary keys             │
│         Tables with single-column primary keys are automatically 2NF       │
│         (if they're in 1NF)                                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Understanding Partial Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PARTIAL DEPENDENCY                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   FULL DEPENDENCY:                                                          │
│   (A, B) → C where neither A alone nor B alone determines C                │
│                                                                              │
│   PARTIAL DEPENDENCY:                                                       │
│   (A, B) → C where A → C OR B → C                                          │
│   (C depends on only PART of the key)                                       │
│                                                                              │
│   Example:                                                                   │
│   Primary Key: (student_id, course_id)                                      │
│                                                                              │
│   FULL: (student_id, course_id) → grade                                    │
│         You need both to determine the grade ✓                              │
│                                                                              │
│   PARTIAL: (student_id, course_id) → student_name                          │
│            student_id alone determines student_name ✗                       │
│                                                                              │
│   PARTIAL: (student_id, course_id) → course_title                          │
│            course_id alone determines course_title ✗                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 2NF Violation Example

### 3.1 Problematic Table

```sql
-- Enrollment table (violates 2NF)
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    student_name VARCHAR(100),    -- Partial dependency on student_id
    student_email VARCHAR(255),   -- Partial dependency on student_id
    course_title VARCHAR(200),    -- Partial dependency on course_id
    course_credits INT,           -- Partial dependency on course_id
    instructor_name VARCHAR(100), -- Partial dependency on course_id
    grade CHAR(2),                -- Full dependency (needs both)
    enrollment_date DATE,         -- Full dependency (needs both)
    PRIMARY KEY (student_id, course_id)
);
```

```
┌────────────┬───────────┬──────────────┬─────────────┬──────────────┬─────────┬────────────────┬───────┬────────────────┐
│ student_id │ course_id │ student_name │student_email│ course_title │ credits │ instructor     │ grade │enrollment_date │
├────────────┼───────────┼──────────────┼─────────────┼──────────────┼─────────┼────────────────┼───────┼────────────────┤
│    101     │   CS101   │    Alice     │alice@u.edu  │ Intro to CS  │    3    │  Dr. Smith     │   A   │  2024-01-15    │
│    101     │   CS102   │    Alice     │alice@u.edu  │ Data Struct  │    3    │  Dr. Jones     │   B   │  2024-01-15    │
│    102     │   CS101   │     Bob      │ bob@u.edu   │ Intro to CS  │    3    │  Dr. Smith     │   A   │  2024-01-16    │
│    102     │   CS201   │     Bob      │ bob@u.edu   │  Algorithms  │    4    │  Dr. Smith     │   C   │  2024-01-16    │
│    103     │   CS101   │    Carol     │carol@u.edu  │ Intro to CS  │    3    │  Dr. Smith     │   B   │  2024-01-17    │
└────────────┴───────────┴──────────────┴─────────────┴──────────────┴─────────┴────────────────┴───────┴────────────────┘

Problems (Anomalies):

REDUNDANCY:
• "Alice" and "alice@u.edu" stored twice (once per course)
• "Intro to CS" and "Dr. Smith" stored three times (once per student)

UPDATE ANOMALY:
• To change Alice's email, must update multiple rows
• To change course title, must update multiple rows
• Risk of inconsistency if not all rows updated

INSERT ANOMALY:
• Can't add a new course until a student enrolls
• Can't add a new student until they enroll in a course

DELETE ANOMALY:
• If Carol drops CS101, we lose all her information
• If all students drop a course, we lose course information
```

---

## 4. Converting to 2NF

### 4.1 Identify Dependencies

```
Primary Key: (student_id, course_id)

Partial Dependencies (violate 2NF):
• student_id → student_name, student_email
• course_id → course_title, course_credits, instructor_name

Full Dependencies (okay):
• (student_id, course_id) → grade, enrollment_date
```

### 4.2 Decompose the Table

```sql
-- Step 1: Create Students table (student_id dependencies)
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100) NOT NULL,
    student_email VARCHAR(255) UNIQUE NOT NULL
);

-- Step 2: Create Courses table (course_id dependencies)
CREATE TABLE courses (
    course_id VARCHAR(10) PRIMARY KEY,
    course_title VARCHAR(200) NOT NULL,
    course_credits INT NOT NULL,
    instructor_name VARCHAR(100)
);

-- Step 3: Keep only full dependencies in Enrollments
CREATE TABLE enrollments (
    student_id INT,
    course_id VARCHAR(10),
    grade CHAR(2),
    enrollment_date DATE NOT NULL,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

### 4.3 Result

```
┌─────────────┐          ┌──────────────────┐          ┌─────────────────────┐
│  students   │          │   enrollments    │          │      courses        │
├─────────────┤          ├──────────────────┤          ├─────────────────────┤
│ student_id  │◄────────┤│ student_id (FK)  │          │ course_id           │
│ student_name│          │ course_id (FK)   │─────────►│ course_title        │
│student_email│          │ grade            │          │ course_credits      │
└─────────────┘          │ enrollment_date  │          │ instructor_name     │
                         └──────────────────┘          └─────────────────────┘

Each student stored ONCE
Each course stored ONCE
Enrollment links them with grade info
```

---

## 5. Another Example: Order Items

### 5.1 Problematic Table

```sql
-- Order Items (violates 2NF)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),    -- Partial: depends only on product_id
    product_category VARCHAR(50), -- Partial: depends only on product_id
    unit_price DECIMAL(10, 2),    -- Could be partial OR full (see below)
    quantity INT,                  -- Full: depends on both
    line_total DECIMAL(10, 2),    -- Full: derived from unit_price * quantity
    PRIMARY KEY (order_id, product_id)
);
```

### 5.2 Price Consideration

```
Is unit_price a partial dependency?

CASE 1: Fixed product prices
• product_id → unit_price
• Partial dependency - violates 2NF
• Solution: Store price in products table

CASE 2: Price can vary by order (discounts, date-based pricing)
• (order_id, product_id) → unit_price
• Full dependency - okay in this table
• Store the actual price charged, not current price

RECOMMENDATION: Store price at time of sale in order_items
(historical accuracy, even if it creates some redundancy)
```

### 5.3 Normalized Version

```sql
-- Products table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    product_category VARCHAR(50),
    current_price DECIMAL(10, 2) NOT NULL
);

-- Order Items (now in 2NF)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,  -- Price at time of purchase
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Note: unit_price is stored here because it captures the historical
-- price at time of order, not the current product price
```

---

## 6. Identifying 2NF Violations

### 6.1 Step-by-Step Process

```
1. Ensure table is in 1NF
2. Identify the primary key
3. If primary key is single column → automatically 2NF
4. If composite primary key:
   a. List all non-key attributes
   b. For each non-key attribute, check:
      - Does it depend on the ENTIRE key?
      - Or just PART of the key?
   c. Attributes depending on part of key = violation
```

### 6.2 SQL Check for Violations

```sql
-- Check if product_name varies for same product_id
-- (indicates product_name should be in products table)
SELECT product_id, COUNT(DISTINCT product_name) as name_count
FROM order_items
GROUP BY product_id
HAVING COUNT(DISTINCT product_name) > 1;

-- If this returns rows, there's data inconsistency
-- If it returns nothing, product_name is functionally dependent on product_id
-- Either way, it's a partial dependency and violates 2NF
```

---

## 7. Benefits of 2NF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      BENEFITS OF 2NF                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ELIMINATES REDUNDANCY                                                     │
│   • Each fact stored once                                                   │
│   • Less storage space                                                      │
│   • No duplicate data to maintain                                           │
│                                                                              │
│   PREVENTS UPDATE ANOMALIES                                                 │
│   • Update in one place                                                     │
│   • No risk of partial updates                                              │
│   • Data stays consistent                                                   │
│                                                                              │
│   PREVENTS INSERT ANOMALIES                                                 │
│   • Can add products without orders                                         │
│   • Can add students without enrollments                                    │
│                                                                              │
│   PREVENTS DELETE ANOMALIES                                                 │
│   • Deleting enrollment doesn't lose student info                          │
│   • Deleting order item doesn't lose product info                          │
│                                                                              │
│   BETTER QUERY PERFORMANCE                                                  │
│   • Smaller tables (less data to scan)                                      │
│   • Better index utilization                                                │
│   • More efficient joins on keys                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Code Example: ORM Approach

```python
# SQLAlchemy example of 2NF design

from sqlalchemy import Column, Integer, String, ForeignKey, Date
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Student(Base):
    __tablename__ = 'students'

    student_id = Column(Integer, primary_key=True)
    student_name = Column(String(100), nullable=False)
    student_email = Column(String(255), unique=True, nullable=False)

    enrollments = relationship('Enrollment', back_populates='student')


class Course(Base):
    __tablename__ = 'courses'

    course_id = Column(String(10), primary_key=True)
    course_title = Column(String(200), nullable=False)
    course_credits = Column(Integer, nullable=False)
    instructor_name = Column(String(100))

    enrollments = relationship('Enrollment', back_populates='course')


class Enrollment(Base):
    __tablename__ = 'enrollments'

    student_id = Column(Integer, ForeignKey('students.student_id'), primary_key=True)
    course_id = Column(String(10), ForeignKey('courses.course_id'), primary_key=True)
    grade = Column(String(2))
    enrollment_date = Column(Date, nullable=False)

    student = relationship('Student', back_populates='enrollments')
    course = relationship('Course', back_populates='enrollments')


# Query example - data is joined, not duplicated
enrollment = session.query(Enrollment).first()
print(f"{enrollment.student.student_name} got {enrollment.grade} in {enrollment.course.course_title}")
```

---

## 9. Summary

| Concept | Description |
|---------|-------------|
| **2NF Requirement** | No partial dependencies on composite primary key |
| **Partial Dependency** | Non-key attribute depends on part of the key |
| **Full Dependency** | Non-key attribute depends on entire key |
| **Single-column PK** | Automatically satisfies 2NF (if in 1NF) |
| **Fix** | Decompose table - move partial dependencies to separate tables |

**Key Rule**: Every non-key attribute must depend on the whole key, not just part of it.

2NF is often naturally achieved when you model entities properly from the start - each "thing" (student, course, product) gets its own table.
