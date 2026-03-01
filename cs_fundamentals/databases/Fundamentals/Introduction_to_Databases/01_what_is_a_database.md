# What is a Database

## 1. Definition

A **database** is an organized collection of structured information, or data, typically stored electronically in a computer system. A database is usually controlled by a **Database Management System (DBMS)**.

### The Database vs File System Problem

Before databases, applications stored data in flat files:

```
# users.txt (Flat File Approach)
1,John,Doe,john@email.com,2023-01-15
2,Jane,Smith,jane@email.com,2023-02-20
3,Bob,Wilson,bob@email.com,2023-03-10
```

**Problems with File Systems:**

| Problem | Description |
|---------|-------------|
| **Data Redundancy** | Same data duplicated across multiple files |
| **Data Inconsistency** | Updates in one file not reflected in others |
| **Difficult Data Access** | No standard way to query data |
| **Isolation Problems** | Data scattered across files in different formats |
| **Integrity Issues** | No enforcement of data rules |
| **Atomicity Problems** | Partial updates can corrupt data |
| **Concurrent Access** | Multiple users can corrupt shared files |
| **Security Issues** | Hard to restrict access to specific data |

---

## 2. Database Management System (DBMS)

A **DBMS** is software that interacts with end users, applications, and the database itself to capture and analyze data. It provides:

```
┌─────────────────────────────────────────────────────────────┐
│                      APPLICATION LAYER                       │
│         (Web Apps, Mobile Apps, Desktop Apps, APIs)          │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                           DBMS                               │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │   Query     │  │  Transaction │  │    Storage        │  │
│  │  Processor  │  │   Manager    │  │    Manager        │  │
│  └─────────────┘  └──────────────┘  └───────────────────┘  │
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │  Security   │  │   Buffer     │  │    Recovery       │  │
│  │   Manager   │  │    Pool      │  │    Manager        │  │
│  └─────────────┘  └──────────────┘  └───────────────────┘  │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      STORAGE LAYER                           │
│              (Data Files, Index Files, Logs)                 │
└─────────────────────────────────────────────────────────────┘
```

### DBMS Components

1. **Query Processor**: Parses, optimizes, and executes queries
2. **Transaction Manager**: Ensures ACID properties
3. **Storage Manager**: Handles disk I/O and file organization
4. **Buffer Pool**: Caches frequently accessed data in memory
5. **Recovery Manager**: Handles crash recovery and data durability

---

## 3. Core Database Terminology

### Schema and Instance

```sql
-- Schema: The STRUCTURE (metadata)
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP
);

-- Instance: The actual DATA at a point in time
-- | id | name  | email           | created_at          |
-- |----|-------|-----------------|---------------------|
-- | 1  | John  | john@email.com  | 2023-01-15 10:30:00 |
-- | 2  | Jane  | jane@email.com  | 2023-02-20 14:45:00 |
```

### Tables, Rows, and Columns

| Term | Also Known As | Description |
|------|---------------|-------------|
| **Table** | Relation | A collection of related data entries |
| **Row** | Record, Tuple | A single entry in a table |
| **Column** | Field, Attribute | A specific piece of data in a row |

### Keys

```sql
-- Primary Key: Uniquely identifies each row
CREATE TABLE products (
    product_id INT PRIMARY KEY,  -- Primary Key
    name VARCHAR(100),
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories(id)  -- Foreign Key
);
```

- **Primary Key**: Unique identifier for each row
- **Foreign Key**: References primary key in another table
- **Candidate Key**: Column(s) that could serve as primary key
- **Composite Key**: Primary key consisting of multiple columns

---

## 4. Database Languages

### DDL - Data Definition Language

Defines database structure:

```sql
-- Create table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department_id INT,
    salary DECIMAL(10, 2)
);

-- Modify structure
ALTER TABLE employees ADD COLUMN hire_date DATE;

-- Remove table
DROP TABLE employees;
```

### DML - Data Manipulation Language

Manipulates data within tables:

```sql
-- Insert data
INSERT INTO employees (id, name, salary) VALUES (1, 'John', 50000);

-- Update data
UPDATE employees SET salary = 55000 WHERE id = 1;

-- Delete data
DELETE FROM employees WHERE id = 1;
```

### DQL - Data Query Language

Retrieves data from tables:

```sql
-- Select data
SELECT name, salary
FROM employees
WHERE department_id = 5
ORDER BY salary DESC;
```

### DCL - Data Control Language

Controls access and permissions:

```sql
-- Grant permissions
GRANT SELECT, INSERT ON employees TO user_role;

-- Revoke permissions
REVOKE DELETE ON employees FROM user_role;
```

### TCL - Transaction Control Language

Manages transactions:

```sql
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Or rollback on error
ROLLBACK;
```

---

## 5. Why Databases Matter

### Data Integrity

```sql
-- Constraints ensure data quality
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    total DECIMAL(10, 2) CHECK (total >= 0),
    status VARCHAR(20) DEFAULT 'pending',
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

### Concurrent Access

Multiple users can safely access data simultaneously:

```
User A: UPDATE inventory SET stock = stock - 1 WHERE product_id = 100;
User B: UPDATE inventory SET stock = stock - 1 WHERE product_id = 100;

-- DBMS ensures both updates are applied correctly using locks/MVCC
```

### Data Recovery

```
┌──────────────────────────────────────────────────────────────┐
│                    RECOVERY PROCESS                           │
│                                                               │
│  1. Transaction Log (WAL) records all changes                │
│  2. Checkpoint saves consistent state periodically           │
│  3. On crash: Replay log from last checkpoint               │
│  4. Undo uncommitted transactions                           │
│  5. Redo committed transactions                             │
└──────────────────────────────────────────────────────────────┘
```

### Performance Optimization

- **Indexes**: Speed up data retrieval
- **Query Optimization**: DBMS chooses efficient execution plans
- **Caching**: Frequently accessed data stays in memory
- **Partitioning**: Distribute data across storage

---

## 6. Database Abstraction Levels

```
┌─────────────────────────────────────────────────────────────┐
│                    VIEW LEVEL (External)                     │
│          What users/applications see                         │
│          View 1    View 2    View 3                         │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                  LOGICAL LEVEL (Conceptual)                  │
│          Complete database structure                         │
│          Tables, relationships, constraints                  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│                  PHYSICAL LEVEL (Internal)                   │
│          How data is actually stored                         │
│          Files, indexes, storage layout                      │
└─────────────────────────────────────────────────────────────┘
```

### Data Independence

- **Logical Data Independence**: Changes to logical schema don't affect external views
- **Physical Data Independence**: Changes to physical storage don't affect logical schema

---

## 7. Code Examples Across Languages

### Python (using sqlite3)

```python
import sqlite3

# Connect to database
conn = sqlite3.connect('example.db')
cursor = conn.cursor()

# Create table
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE
    )
''')

# Insert data
cursor.execute("INSERT INTO users (name, email) VALUES (?, ?)",
               ("John Doe", "john@example.com"))

# Query data
cursor.execute("SELECT * FROM users WHERE name LIKE ?", ("%John%",))
rows = cursor.fetchall()
for row in rows:
    print(f"ID: {row[0]}, Name: {row[1]}, Email: {row[2]}")

conn.commit()
conn.close()
```

### Java (using JDBC)

```java
import java.sql.*;

public class DatabaseExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mydb";
        String user = "root";
        String password = "password";

        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            // Create table
            String createTable = """
                CREATE TABLE IF NOT EXISTS users (
                    id INT PRIMARY KEY AUTO_INCREMENT,
                    name VARCHAR(100) NOT NULL,
                    email VARCHAR(255) UNIQUE
                )
            """;
            conn.createStatement().execute(createTable);

            // Insert data with prepared statement
            String insert = "INSERT INTO users (name, email) VALUES (?, ?)";
            try (PreparedStatement pstmt = conn.prepareStatement(insert)) {
                pstmt.setString(1, "John Doe");
                pstmt.setString(2, "john@example.com");
                pstmt.executeUpdate();
            }

            // Query data
            String query = "SELECT * FROM users WHERE name LIKE ?";
            try (PreparedStatement pstmt = conn.prepareStatement(query)) {
                pstmt.setString(1, "%John%");
                ResultSet rs = pstmt.executeQuery();
                while (rs.next()) {
                    System.out.printf("ID: %d, Name: %s, Email: %s%n",
                        rs.getInt("id"),
                        rs.getString("name"),
                        rs.getString("email"));
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### JavaScript (using node-postgres)

```javascript
const { Pool } = require('pg');

const pool = new Pool({
    host: 'localhost',
    database: 'mydb',
    user: 'postgres',
    password: 'password',
    port: 5432,
});

async function main() {
    const client = await pool.connect();

    try {
        // Create table
        await client.query(`
            CREATE TABLE IF NOT EXISTS users (
                id SERIAL PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                email VARCHAR(255) UNIQUE
            )
        `);

        // Insert data with parameterized query
        await client.query(
            'INSERT INTO users (name, email) VALUES ($1, $2)',
            ['John Doe', 'john@example.com']
        );

        // Query data
        const result = await client.query(
            'SELECT * FROM users WHERE name LIKE $1',
            ['%John%']
        );

        result.rows.forEach(row => {
            console.log(`ID: ${row.id}, Name: ${row.name}, Email: ${row.email}`);
        });
    } finally {
        client.release();
    }

    await pool.end();
}

main().catch(console.error);
```

---

## 8. Summary

| Concept | Description |
|---------|-------------|
| **Database** | Organized collection of structured data |
| **DBMS** | Software that manages database operations |
| **Schema** | Structure definition (tables, columns, constraints) |
| **Instance** | Actual data at a specific point in time |
| **ACID** | Properties ensuring reliable transactions |
| **SQL** | Standard language for database operations |

Databases solve the fundamental problems of data storage, retrieval, integrity, and concurrent access that file systems cannot handle efficiently.
