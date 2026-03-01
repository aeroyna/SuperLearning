# BCNF and Higher Normal Forms

## 1. Introduction

**Boyce-Codd Normal Form (BCNF)** is a stricter version of 3NF. Beyond BCNF, there are 4NF and 5NF which address more complex dependency issues.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NORMAL FORMS HIERARCHY                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1NF  →  2NF  →  3NF  →  BCNF  →  4NF  →  5NF                             │
│   ↑        ↑       ↑       ↑        ↑       ↑                               │
│   Atomic   No      No      Every    No      No                              │
│   values   partial trans.  det. is  multi   join                            │
│            deps    deps    super    valued  deps                             │
│                            key      deps                                     │
│                                                                              │
│   Most databases aim for 3NF or BCNF                                        │
│   4NF and 5NF are rarely needed in practice                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Boyce-Codd Normal Form (BCNF)

### 2.1 Definition

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BCNF DEFINITION                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A table is in BCNF if:                                                    │
│                                                                              │
│   For every non-trivial functional dependency X → Y:                       │
│   • X must be a SUPERKEY                                                   │
│                                                                              │
│   (No exceptions for prime attributes like in 3NF)                          │
│                                                                              │
│   Comparison:                                                               │
│   3NF allows: X → A where A is a prime attribute (part of candidate key)   │
│   BCNF: X must ALWAYS be a superkey, no exceptions                          │
│                                                                              │
│   BCNF is stricter than 3NF, but every BCNF table is also in 3NF           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 3NF vs BCNF Example

```sql
-- Consider a student-subject-teacher scenario
-- Rule: A teacher teaches only one subject
-- Rule: Multiple teachers can teach the same subject to a student

-- Table (violates BCNF but satisfies 3NF):
CREATE TABLE enrollments (
    student_id INT,
    subject VARCHAR(50),
    teacher_id INT,
    PRIMARY KEY (student_id, subject)
);
```

```
┌────────────┬───────────┬────────────┐
│ student_id │  subject  │ teacher_id │
├────────────┼───────────┼────────────┤
│    101     │   Math    │     T1     │
│    101     │  Physics  │     T2     │
│    102     │   Math    │     T1     │
│    102     │  Physics  │     T3     │
│    103     │   Math    │     T4     │
└────────────┴───────────┴────────────┘

Candidate Keys: (student_id, subject)
                (student_id, teacher_id) - because teacher → subject

Functional Dependencies:
• (student_id, subject) → teacher_id
• teacher_id → subject                  ← PROBLEM!

teacher_id → subject violates BCNF:
• teacher_id is NOT a superkey
• But it's allowed in 3NF because subject is a prime attribute

This IS in 3NF (subject is part of a candidate key)
This is NOT in BCNF (teacher_id is not a superkey)
```

### 2.3 Converting to BCNF

```sql
-- Decompose the table
-- Separate the problematic dependency: teacher_id → subject

CREATE TABLE teachers (
    teacher_id INT PRIMARY KEY,
    subject VARCHAR(50) NOT NULL
);

CREATE TABLE student_teachers (
    student_id INT,
    teacher_id INT,
    PRIMARY KEY (student_id, teacher_id),
    FOREIGN KEY (teacher_id) REFERENCES teachers(teacher_id)
);

-- Now both tables are in BCNF:
-- teachers: teacher_id is the key, determines everything
-- student_teachers: (student_id, teacher_id) is the key
```

```
┌───────────────────┐            ┌──────────────────────┐
│     teachers      │            │   student_teachers   │
├───────────────────┤            ├──────────────────────┤
│ teacher_id (PK)   │◄───────────│ teacher_id (FK, PK)  │
│ subject           │            │ student_id (PK)      │
└───────────────────┘            └──────────────────────┘
```

### 2.4 Trade-off: BCNF May Lose Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BCNF TRADE-OFF                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Original FD: (student_id, subject) → teacher_id                          │
│                                                                              │
│   After BCNF decomposition, we CAN'T enforce this without a trigger!       │
│                                                                              │
│   The decomposed tables can have:                                           │
│   • Student 101 with teachers T1 and T2 (both teach Math)                  │
│   • We can no longer enforce "one teacher per subject per student"         │
│                                                                              │
│   DECISION:                                                                 │
│   • If dependency enforcement is critical → Stay at 3NF                    │
│   • If data integrity via FDs isn't needed → Use BCNF                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Fourth Normal Form (4NF)

### 3.1 Multi-Valued Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MULTI-VALUED DEPENDENCY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   X →→ Y (X multi-determines Y) means:                                      │
│   For a given X value, the set of Y values is independent of other         │
│   attributes.                                                                │
│                                                                              │
│   Example: An employee can have multiple skills AND multiple languages     │
│   • Employee 101: Skills = {Java, Python}, Languages = {English, Spanish}  │
│                                                                              │
│   If we store in one table with all combinations:                           │
│   ┌──────────┬─────────┬──────────┐                                         │
│   │ emp_id   │  skill  │ language │                                         │
│   ├──────────┼─────────┼──────────┤                                         │
│   │   101    │  Java   │ English  │                                         │
│   │   101    │  Java   │ Spanish  │                                         │
│   │   101    │ Python  │ English  │                                         │
│   │   101    │ Python  │ Spanish  │                                         │
│   └──────────┴─────────┴──────────┘                                         │
│                                                                              │
│   Redundancy! Skills and languages are independent.                         │
│   emp_id →→ skill and emp_id →→ language                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 4NF Definition

```
A table is in 4NF if:
1. It is in BCNF
2. For every non-trivial multi-valued dependency X →→ Y:
   • X is a superkey

Translation: A table should not contain two or more independent
multi-valued facts about an entity.
```

### 3.3 Converting to 4NF

```sql
-- Violates 4NF
CREATE TABLE emp_skills_languages (
    emp_id INT,
    skill VARCHAR(50),
    language VARCHAR(50),
    PRIMARY KEY (emp_id, skill, language)
);

-- 4NF Solution: Separate independent multi-valued attributes
CREATE TABLE emp_skills (
    emp_id INT,
    skill VARCHAR(50),
    PRIMARY KEY (emp_id, skill)
);

CREATE TABLE emp_languages (
    emp_id INT,
    language VARCHAR(50),
    PRIMARY KEY (emp_id, language)
);

-- Now no redundancy:
-- emp_skills: (101, Java), (101, Python)
-- emp_languages: (101, English), (101, Spanish)
```

---

## 4. Fifth Normal Form (5NF)

### 4.1 Join Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     JOIN DEPENDENCY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A table has a join dependency *(R1, R2, ..., Rn) if it can be            │
│   reconstructed by joining projections on R1, R2, ..., Rn                  │
│                                                                              │
│   5NF: Every join dependency is implied by candidate keys                   │
│                                                                              │
│   This is very rare in practice and often theoretical                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 5NF Example

```sql
-- Consider: Suppliers, Parts, and Projects
-- Rule: A supplier supplies a part to a project ONLY if:
--       1. Supplier supplies the part (to someone)
--       2. Supplier supplies to the project (something)
--       3. The part is used in the project (by someone)

-- Violates 5NF if all three binary relationships exist
CREATE TABLE supplies (
    supplier_id INT,
    part_id INT,
    project_id INT,
    PRIMARY KEY (supplier_id, part_id, project_id)
);

-- 5NF decomposition (if the business rule applies):
CREATE TABLE supplier_parts (
    supplier_id INT,
    part_id INT,
    PRIMARY KEY (supplier_id, part_id)
);

CREATE TABLE supplier_projects (
    supplier_id INT,
    project_id INT,
    PRIMARY KEY (supplier_id, project_id)
);

CREATE TABLE part_projects (
    part_id INT,
    project_id INT,
    PRIMARY KEY (part_id, project_id)
);

-- Original can be reconstructed by joining all three
-- But only if the specific business rule holds!
```

---

## 5. Practical Guidelines

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   WHICH NORMAL FORM TO USE?                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   MOST APPLICATIONS: 3NF or BCNF                                            │
│   • Eliminates most redundancy                                              │
│   • Prevents most anomalies                                                 │
│   • Practical to implement and maintain                                     │
│                                                                              │
│   USE 3NF WHEN:                                                             │
│   • BCNF decomposition would lose important dependencies                    │
│   • Need to enforce complex constraints via FDs                             │
│                                                                              │
│   USE BCNF WHEN:                                                            │
│   • 3NF still has problematic redundancy                                    │
│   • Can enforce lost dependencies via application/triggers                 │
│                                                                              │
│   USE 4NF WHEN:                                                             │
│   • You have independent multi-valued attributes                           │
│   • Seeing significant storage waste from combinations                      │
│                                                                              │
│   USE 5NF WHEN:                                                             │
│   • Very rare, complex scenarios with specific business rules              │
│   • Usually only in academic/specialized contexts                          │
│                                                                              │
│   DENORMALIZE WHEN:                                                         │
│   • Read performance is critical                                           │
│   • Data is mostly read, rarely updated                                    │
│   • Have proper update mechanisms                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Algorithm: Decomposition to BCNF

```python
def decompose_to_bcnf(relation, fds):
    """
    Decompose a relation to BCNF.

    Args:
        relation: set of attributes
        fds: list of functional dependencies (lhs, rhs)

    Returns:
        list of BCNF relations
    """
    result = [relation]

    while True:
        found_violation = False

        for r in result:
            for lhs, rhs in fds:
                # Check if FD applies to this relation
                if not lhs.issubset(r) or not rhs.issubset(r):
                    continue

                # Check if LHS is a superkey of r
                closure = compute_closure(lhs, fds)
                if not r.issubset(closure):
                    # FD violates BCNF
                    found_violation = True

                    # Decompose: R1 = lhs ∪ rhs, R2 = R - rhs + lhs
                    r1 = lhs.union(rhs)
                    r2 = r.difference(rhs).union(lhs)

                    result.remove(r)
                    result.extend([r1, r2])
                    break

            if found_violation:
                break

        if not found_violation:
            break

    return result

# Example usage:
# relation = {'A', 'B', 'C', 'D'}
# fds = [({'A'}, {'B'}), ({'C'}, {'D'})]
# result = decompose_to_bcnf(relation, fds)
# Result: [{'A', 'B'}, {'C', 'D'}, {'A', 'C'}]
```

---

## 7. Identifying Normal Form Violations

```sql
-- Check for potential BCNF violations
-- Find columns that determine others but aren't unique

-- Step 1: Find columns with functional dependencies
SELECT column_a, COUNT(DISTINCT column_b) as b_count
FROM my_table
GROUP BY column_a
HAVING COUNT(DISTINCT column_b) = 1 AND COUNT(*) > 1;

-- If column_a always maps to one column_b value,
-- but column_a is not a key, this might indicate a BCNF violation

-- Step 2: Check if the determining column is a candidate key
-- If NOT, it's a BCNF violation

-- Example check for teacher_id → subject
SELECT teacher_id, COUNT(DISTINCT subject) as subject_count
FROM enrollments
GROUP BY teacher_id;
-- If all counts are 1, teacher_id → subject
-- If teacher_id is not a key, BCNF is violated
```

---

## 8. Summary

| Normal Form | Requirement | Eliminates |
|-------------|-------------|------------|
| **BCNF** | Every determinant is a superkey | All FD-based redundancy |
| **4NF** | No non-trivial multi-valued dependencies | Independent multi-valued attribute redundancy |
| **5NF** | All join dependencies implied by keys | Complex join-based redundancy |

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         KEY TAKEAWAYS                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. BCNF is stricter than 3NF - every determinant must be a superkey      │
│                                                                              │
│   2. BCNF decomposition may lose the ability to enforce some FDs           │
│                                                                              │
│   3. 4NF handles independent multi-valued attributes                       │
│                                                                              │
│   4. 5NF is rarely needed in practice                                      │
│                                                                              │
│   5. For most applications, 3NF or BCNF is sufficient                      │
│                                                                              │
│   6. Always consider the trade-offs between normalization and performance  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```
