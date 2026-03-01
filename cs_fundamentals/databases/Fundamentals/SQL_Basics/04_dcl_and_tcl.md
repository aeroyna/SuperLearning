# DCL and TCL - Data Control & Transaction Control Language

## 1. Introduction

**DCL (Data Control Language)** manages permissions and access control, while **TCL (Transaction Control Language)** manages database transactions.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DCL AND TCL OVERVIEW                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   DCL - Data Control Language:                                              │
│   • GRANT   - Give permissions to users/roles                               │
│   • REVOKE  - Remove permissions from users/roles                           │
│                                                                              │
│   TCL - Transaction Control Language:                                       │
│   • BEGIN / START TRANSACTION  - Start a transaction                        │
│   • COMMIT    - Save all changes                                            │
│   • ROLLBACK  - Undo all changes                                            │
│   • SAVEPOINT - Create a point to rollback to                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# Part 1: DCL - Data Control Language

## 2. Users and Roles

### 2.1 Creating Users

```sql
-- PostgreSQL
CREATE USER app_user WITH PASSWORD 'secure_password';
CREATE USER readonly_user WITH PASSWORD 'password123' VALID UNTIL '2025-12-31';

-- MySQL
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'secure_password';
CREATE USER 'app_user'@'%' IDENTIFIED BY 'secure_password';  -- Any host

-- SQL Server
CREATE LOGIN app_user WITH PASSWORD = 'secure_password';
CREATE USER app_user FOR LOGIN app_user;

-- Oracle
CREATE USER app_user IDENTIFIED BY secure_password;
```

### 2.2 Creating Roles

```sql
-- PostgreSQL
CREATE ROLE developers;
CREATE ROLE analysts;
CREATE ROLE admins WITH CREATEDB CREATEROLE;

-- MySQL
CREATE ROLE 'developers', 'analysts', 'admins';

-- SQL Server
CREATE ROLE developers;
CREATE ROLE analysts;

-- Add users to roles
-- PostgreSQL
GRANT developers TO app_user;
GRANT analysts TO readonly_user;

-- MySQL
GRANT 'developers' TO 'app_user'@'localhost';

-- SQL Server
ALTER ROLE developers ADD MEMBER app_user;
```

---

## 3. GRANT - Giving Permissions

### 3.1 Table Privileges

```sql
-- Grant specific privileges on a table
GRANT SELECT ON employees TO readonly_user;
GRANT SELECT, INSERT ON employees TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO developers;

-- Grant all privileges
GRANT ALL PRIVILEGES ON employees TO admins;
GRANT ALL ON employees TO admins;  -- Shorthand

-- Grant to multiple users
GRANT SELECT ON employees TO user1, user2, user3;

-- Grant on all tables in schema (PostgreSQL)
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- Grant on all tables (MySQL)
GRANT SELECT ON database_name.* TO 'readonly_user'@'localhost';
```

### 3.2 Column-Level Privileges

```sql
-- Grant access to specific columns only
GRANT SELECT (first_name, last_name, department_id) ON employees TO readonly_user;
GRANT UPDATE (salary) ON employees TO hr_user;

-- User can only see/update specified columns
-- Trying to access other columns will fail
```

### 3.3 Privilege Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRIVILEGE TYPES                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   TABLE PRIVILEGES:                                                          │
│   • SELECT    - Read data                                                   │
│   • INSERT    - Add new rows                                                │
│   • UPDATE    - Modify existing rows                                        │
│   • DELETE    - Remove rows                                                 │
│   • TRUNCATE  - Empty table (PostgreSQL)                                    │
│   • REFERENCES - Create foreign keys                                        │
│   • TRIGGER   - Create triggers                                             │
│                                                                              │
│   DATABASE PRIVILEGES:                                                       │
│   • CREATE    - Create new objects                                          │
│   • CONNECT   - Connect to database                                         │
│   • TEMPORARY - Create temp tables                                          │
│                                                                              │
│   SCHEMA PRIVILEGES (PostgreSQL):                                           │
│   • USAGE     - Access objects in schema                                    │
│   • CREATE    - Create objects in schema                                    │
│                                                                              │
│   PROCEDURE PRIVILEGES:                                                      │
│   • EXECUTE   - Run stored procedure/function                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.4 Database and Schema Privileges

```sql
-- PostgreSQL
-- Grant connect to database
GRANT CONNECT ON DATABASE myapp TO app_user;

-- Grant schema usage (required to see tables)
GRANT USAGE ON SCHEMA public TO app_user;
GRANT USAGE ON SCHEMA sales TO sales_team;

-- Grant create in schema
GRANT CREATE ON SCHEMA public TO developers;

-- Grant sequence usage
GRANT USAGE ON SEQUENCE employees_id_seq TO app_user;
GRANT ALL ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- MySQL
GRANT ALL PRIVILEGES ON myapp.* TO 'app_user'@'localhost';
GRANT SELECT ON myapp.* TO 'readonly_user'@'localhost';
```

### 3.5 WITH GRANT OPTION

```sql
-- Allow user to grant their privileges to others
GRANT SELECT ON employees TO team_lead WITH GRANT OPTION;

-- team_lead can now run:
-- GRANT SELECT ON employees TO new_team_member;

-- PostgreSQL - Grant role with admin option
GRANT developers TO team_lead WITH ADMIN OPTION;
```

---

## 4. REVOKE - Removing Permissions

### 4.1 Basic REVOKE

```sql
-- Revoke specific privileges
REVOKE SELECT ON employees FROM readonly_user;
REVOKE INSERT, UPDATE ON employees FROM app_user;

-- Revoke all privileges
REVOKE ALL PRIVILEGES ON employees FROM app_user;
REVOKE ALL ON employees FROM app_user;

-- Revoke from role
REVOKE SELECT ON employees FROM developers;

-- Revoke role from user
REVOKE developers FROM app_user;
```

### 4.2 Cascading Revoke

```sql
-- When revoking from someone who granted to others
-- CASCADE - Also revoke from anyone they granted to
REVOKE SELECT ON employees FROM team_lead CASCADE;

-- RESTRICT - Fail if they granted to others (default in PostgreSQL)
REVOKE SELECT ON employees FROM team_lead RESTRICT;
```

### 4.3 Revoke Grant Option

```sql
-- Remove ability to grant, but keep the privilege
REVOKE GRANT OPTION FOR SELECT ON employees FROM team_lead;
```

---

## 5. Viewing Permissions

### 5.1 PostgreSQL

```sql
-- View table privileges
SELECT
    grantee,
    table_schema,
    table_name,
    privilege_type
FROM information_schema.role_table_grants
WHERE table_name = 'employees';

-- View all privileges for a user
SELECT * FROM information_schema.role_table_grants
WHERE grantee = 'app_user';

-- Using psql commands
\dp employees        -- Show table access privileges
\du                  -- List roles
\du+ username        -- Show role details
```

### 5.2 MySQL

```sql
-- Show grants for current user
SHOW GRANTS;

-- Show grants for specific user
SHOW GRANTS FOR 'app_user'@'localhost';

-- Query information_schema
SELECT * FROM information_schema.user_privileges
WHERE grantee LIKE '%app_user%';

SELECT * FROM information_schema.table_privileges
WHERE grantee LIKE '%app_user%';
```

### 5.3 SQL Server

```sql
-- View permissions on a table
SELECT
    pr.principal_id,
    pr.name AS principal_name,
    pr.type_desc,
    pe.state_desc,
    pe.permission_name
FROM sys.database_permissions pe
JOIN sys.database_principals pr ON pe.grantee_principal_id = pr.principal_id
WHERE pe.major_id = OBJECT_ID('employees');
```

---

## 6. Best Practices for DCL

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    DCL BEST PRACTICES                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. PRINCIPLE OF LEAST PRIVILEGE                                           │
│      • Grant only necessary permissions                                     │
│      • Start with minimal access, add as needed                            │
│      • Regularly audit and remove unused permissions                        │
│                                                                              │
│   2. USE ROLES                                                               │
│      • Create roles for common permission sets                             │
│      • Assign users to roles, not individual permissions                   │
│      • Easier to manage and audit                                          │
│                                                                              │
│   3. AVOID GRANTING TO PUBLIC                                               │
│      • Don't: GRANT SELECT ON sensitive_data TO PUBLIC                     │
│      • Revoke default public permissions if needed                         │
│                                                                              │
│   4. SEPARATE READ AND WRITE                                                │
│      • Create separate roles for reading vs modifying                      │
│      • readonly_role, readwrite_role, admin_role                           │
│                                                                              │
│   5. APPLICATION USERS                                                       │
│      • Create dedicated users for applications                             │
│      • Don't use admin accounts in application code                        │
│      • Different users for different services                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# Part 2: TCL - Transaction Control Language

## 7. Understanding Transactions

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      WHAT IS A TRANSACTION?                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   A transaction is a sequence of operations performed as a single           │
│   logical unit of work. It must satisfy ACID properties:                    │
│                                                                              │
│   • Atomicity   - All or nothing                                           │
│   • Consistency - Valid state to valid state                               │
│   • Isolation   - Concurrent transactions don't interfere                  │
│   • Durability  - Committed changes are permanent                          │
│                                                                              │
│   Example: Bank Transfer                                                     │
│   ┌───────────────────────────────────────────────────────────────┐        │
│   │  BEGIN TRANSACTION                                             │        │
│   │      UPDATE accounts SET balance = balance - 100 WHERE id=1   │        │
│   │      UPDATE accounts SET balance = balance + 100 WHERE id=2   │        │
│   │  COMMIT                                                        │        │
│   └───────────────────────────────────────────────────────────────┘        │
│                                                                              │
│   Both updates succeed together, or neither happens                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. BEGIN / START TRANSACTION

```sql
-- PostgreSQL
BEGIN;
-- or
BEGIN TRANSACTION;
-- or
START TRANSACTION;

-- MySQL
START TRANSACTION;
-- or
BEGIN;

-- SQL Server
BEGIN TRANSACTION;
-- or
BEGIN TRAN;

-- Transaction with options (PostgreSQL)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION READ ONLY;
```

---

## 9. COMMIT - Saving Changes

```sql
-- Save all changes made in the transaction
BEGIN;
    INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
    INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 5, 2);
    UPDATE inventory SET quantity = quantity - 2 WHERE product_id = 5;
COMMIT;

-- All three statements are now permanent
-- Other transactions can now see these changes
```

---

## 10. ROLLBACK - Undoing Changes

```sql
-- Undo all changes made in the transaction
BEGIN;
    DELETE FROM employees WHERE department_id = 10;
    -- Oops, wrong department!
ROLLBACK;

-- No employees were deleted

-- Error handling example
BEGIN;
    INSERT INTO orders (customer_id, total) VALUES (1, 100.00);
    -- This might fail (e.g., constraint violation)
    INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 999, 2);
ROLLBACK;  -- Undo everything if there's an error
```

---

## 11. SAVEPOINT - Partial Rollback

```sql
-- Create named points to rollback to
BEGIN;
    INSERT INTO audit_log (action) VALUES ('Starting batch process');
    SAVEPOINT after_audit;

    UPDATE products SET price = price * 1.1 WHERE category = 'electronics';
    SAVEPOINT after_electronics;

    UPDATE products SET price = price * 1.1 WHERE category = 'furniture';
    -- Oops, furniture prices shouldn't change

    ROLLBACK TO SAVEPOINT after_electronics;
    -- Furniture update is undone, electronics update is kept

COMMIT;
-- audit_log insert and electronics update are committed

-- Release savepoint (optional cleanup)
RELEASE SAVEPOINT after_electronics;
```

### 11.1 Savepoint Use Cases

```sql
-- Batch processing with error recovery
BEGIN;
    SAVEPOINT batch_start;

    FOR each_record IN batch_records LOOP
        SAVEPOINT before_record;

        BEGIN
            -- Process record
            INSERT INTO processed_data VALUES (...);
        EXCEPTION WHEN OTHERS THEN
            -- If single record fails, rollback just that record
            ROLLBACK TO SAVEPOINT before_record;
            -- Log the error and continue
            INSERT INTO error_log VALUES (each_record.id, SQLERRM);
        END;
    END LOOP;

COMMIT;  -- Commit all successful records
```

---

## 12. Autocommit Mode

```sql
-- By default, many databases run in autocommit mode
-- Each statement is its own transaction

-- Disable autocommit (varies by database and client)

-- PostgreSQL (psql)
\set AUTOCOMMIT off

-- MySQL
SET autocommit = 0;  -- Disable
SET autocommit = 1;  -- Enable (default)

-- Check autocommit status
SHOW autocommit;  -- MySQL

-- When autocommit is off, you must explicitly COMMIT
UPDATE products SET price = 10 WHERE id = 1;
COMMIT;  -- Required to make change permanent
```

---

## 13. Transaction Examples

### 13.1 Bank Transfer

```sql
-- Transfer $500 from account 1 to account 2
BEGIN;
    -- Check sufficient funds
    SELECT balance INTO @source_balance FROM accounts WHERE id = 1 FOR UPDATE;

    IF @source_balance < 500 THEN
        ROLLBACK;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient funds';
    END IF;

    -- Perform transfer
    UPDATE accounts SET balance = balance - 500 WHERE id = 1;
    UPDATE accounts SET balance = balance + 500 WHERE id = 2;

    -- Log the transaction
    INSERT INTO transfers (from_account, to_account, amount, transfer_date)
    VALUES (1, 2, 500, NOW());

COMMIT;
```

### 13.2 Inventory Management

```sql
BEGIN;
    -- Reserve inventory
    UPDATE inventory
    SET reserved = reserved + 5,
        available = available - 5
    WHERE product_id = 100 AND available >= 5;

    -- Check if update succeeded
    IF ROW_COUNT() = 0 THEN
        ROLLBACK;
        -- Handle out of stock
    ELSE
        -- Create order
        INSERT INTO orders (product_id, quantity, status)
        VALUES (100, 5, 'pending');
        COMMIT;
    END IF;
```

### 13.3 Multi-Table Insert

```sql
BEGIN;
    -- Create customer
    INSERT INTO customers (name, email) VALUES ('John Doe', 'john@email.com');
    SET @customer_id = LAST_INSERT_ID();  -- MySQL

    -- Create address
    INSERT INTO addresses (customer_id, street, city)
    VALUES (@customer_id, '123 Main St', 'New York');

    -- Create initial order
    INSERT INTO orders (customer_id, status)
    VALUES (@customer_id, 'new');

COMMIT;
-- All three inserts succeed or fail together
```

---

## 14. Error Handling in Transactions

### 14.1 PostgreSQL

```sql
DO $$
BEGIN
    BEGIN
        INSERT INTO orders VALUES (1, 100);
        INSERT INTO order_items VALUES (1, 1, 5);  -- Might fail
    EXCEPTION
        WHEN foreign_key_violation THEN
            RAISE NOTICE 'Foreign key error, rolling back';
            -- Transaction automatically rolled back
        WHEN OTHERS THEN
            RAISE NOTICE 'Unknown error: %', SQLERRM;
    END;
END $$;
```

### 14.2 MySQL

```sql
DELIMITER //
CREATE PROCEDURE safe_transfer(
    IN from_id INT,
    IN to_id INT,
    IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'Transfer failed' AS result;
    END;

    START TRANSACTION;
        UPDATE accounts SET balance = balance - amount WHERE id = from_id;
        UPDATE accounts SET balance = balance + amount WHERE id = to_id;
    COMMIT;

    SELECT 'Transfer successful' AS result;
END //
DELIMITER ;
```

### 14.3 Application-Level (Python)

```python
import psycopg2

try:
    conn = psycopg2.connect(database="mydb")
    cur = conn.cursor()

    cur.execute("BEGIN")
    cur.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
    cur.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
    cur.execute("COMMIT")

except psycopg2.Error as e:
    conn.rollback()
    print(f"Transaction failed: {e}")
finally:
    cur.close()
    conn.close()

# Or using context manager
with psycopg2.connect(database="mydb") as conn:
    with conn.cursor() as cur:
        cur.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
        cur.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
    # Commits automatically on successful exit
    # Rolls back on exception
```

---

## 15. Summary

### DCL Commands

| Command | Purpose | Example |
|---------|---------|---------|
| GRANT | Give permissions | `GRANT SELECT ON table TO user` |
| REVOKE | Remove permissions | `REVOKE INSERT ON table FROM user` |
| CREATE USER | Create database user | `CREATE USER name WITH PASSWORD 'pass'` |
| CREATE ROLE | Create permission group | `CREATE ROLE developers` |

### TCL Commands

| Command | Purpose | Example |
|---------|---------|---------|
| BEGIN | Start transaction | `BEGIN TRANSACTION` |
| COMMIT | Save changes | `COMMIT` |
| ROLLBACK | Undo changes | `ROLLBACK` |
| SAVEPOINT | Create restore point | `SAVEPOINT sp1` |
| ROLLBACK TO | Partial rollback | `ROLLBACK TO SAVEPOINT sp1` |

**Key Points:**
- Use roles instead of granting directly to users
- Apply principle of least privilege
- Always use transactions for related changes
- Handle errors properly with rollback
- Use savepoints for complex operations
