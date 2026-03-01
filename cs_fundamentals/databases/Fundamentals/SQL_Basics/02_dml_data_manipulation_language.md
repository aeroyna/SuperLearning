# DML - Data Manipulation Language

## 1. Introduction

**Data Manipulation Language (DML)** consists of SQL commands used to insert, update, delete, and manage data within database tables. Unlike DDL, DML statements can be rolled back within transactions.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DML COMMANDS                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   INSERT   - Add new rows to a table                                        │
│   UPDATE   - Modify existing rows                                           │
│   DELETE   - Remove rows from a table                                       │
│   MERGE    - Upsert (INSERT or UPDATE based on condition)                   │
│                                                                              │
│   DML statements are transactional - can be committed or rolled back        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. INSERT Statements

### 2.1 Basic INSERT

```sql
-- Insert single row with all columns
INSERT INTO users (id, username, email, created_at)
VALUES (1, 'johndoe', 'john@example.com', NOW());

-- Insert with default values
INSERT INTO users (username, email)
VALUES ('janedoe', 'jane@example.com');

-- Insert without column list (not recommended - fragile)
INSERT INTO users
VALUES (2, 'bobsmith', 'bob@example.com', '2024-01-15 10:30:00');
```

### 2.2 Insert Multiple Rows

```sql
-- Insert multiple rows in one statement
INSERT INTO products (name, price, category)
VALUES
    ('Widget A', 19.99, 'Electronics'),
    ('Widget B', 29.99, 'Electronics'),
    ('Gadget X', 49.99, 'Gadgets'),
    ('Gadget Y', 59.99, 'Gadgets');

-- More efficient than multiple single inserts
-- Reduces network round-trips and transaction overhead
```

### 2.3 INSERT ... SELECT

```sql
-- Insert from another table
INSERT INTO orders_archive (id, customer_id, total, status, created_at)
SELECT id, customer_id, total, status, created_at
FROM orders
WHERE created_at < '2023-01-01';

-- Insert with transformation
INSERT INTO user_stats (user_id, total_orders, total_spent)
SELECT
    customer_id,
    COUNT(*),
    SUM(total)
FROM orders
GROUP BY customer_id;

-- Insert from multiple tables with JOIN
INSERT INTO order_summary (order_id, customer_name, product_names, total)
SELECT
    o.id,
    c.name,
    STRING_AGG(p.name, ', '),
    o.total
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
GROUP BY o.id, c.name, o.total;
```

### 2.4 INSERT with RETURNING (PostgreSQL)

```sql
-- Get inserted row data
INSERT INTO users (username, email)
VALUES ('newuser', 'new@example.com')
RETURNING id, created_at;

-- Return multiple columns
INSERT INTO orders (customer_id, total)
VALUES (1, 99.99)
RETURNING id, order_number, created_at;

-- Use in CTE
WITH new_order AS (
    INSERT INTO orders (customer_id, total)
    VALUES (1, 99.99)
    RETURNING id
)
INSERT INTO order_items (order_id, product_id, quantity)
SELECT id, 101, 2 FROM new_order;
```

### 2.5 INSERT ... ON CONFLICT (UPSERT)

```sql
-- PostgreSQL UPSERT
INSERT INTO users (email, username, login_count)
VALUES ('john@example.com', 'johndoe', 1)
ON CONFLICT (email) DO UPDATE SET
    login_count = users.login_count + 1,
    last_login = NOW();

-- Do nothing on conflict
INSERT INTO users (email, username)
VALUES ('john@example.com', 'johndoe')
ON CONFLICT (email) DO NOTHING;

-- Conflict on composite key
INSERT INTO inventory (product_id, warehouse_id, quantity)
VALUES (1, 1, 100)
ON CONFLICT (product_id, warehouse_id) DO UPDATE SET
    quantity = inventory.quantity + EXCLUDED.quantity;

-- MySQL UPSERT
INSERT INTO users (email, username, login_count)
VALUES ('john@example.com', 'johndoe', 1)
ON DUPLICATE KEY UPDATE
    login_count = login_count + 1,
    last_login = NOW();
```

---

## 3. UPDATE Statements

### 3.1 Basic UPDATE

```sql
-- Update single column
UPDATE users
SET status = 'inactive'
WHERE id = 1;

-- Update multiple columns
UPDATE users
SET
    status = 'active',
    last_login = NOW(),
    login_count = login_count + 1
WHERE email = 'john@example.com';

-- Update all rows (DANGEROUS - no WHERE clause)
UPDATE products
SET price = price * 1.1;  -- 10% price increase
```

### 3.2 Conditional UPDATE

```sql
-- Update with CASE
UPDATE products SET
    price = CASE
        WHEN category = 'Electronics' THEN price * 1.15
        WHEN category = 'Clothing' THEN price * 1.10
        ELSE price * 1.05
    END;

-- Update with subquery condition
UPDATE orders SET
    status = 'cancelled'
WHERE customer_id IN (
    SELECT id FROM customers WHERE status = 'banned'
);
```

### 3.3 UPDATE with JOIN

```sql
-- PostgreSQL: UPDATE FROM
UPDATE orders o
SET status = 'priority'
FROM customers c
WHERE o.customer_id = c.id
  AND c.tier = 'premium';

-- MySQL: UPDATE JOIN
UPDATE orders o
JOIN customers c ON o.customer_id = c.id
SET o.status = 'priority'
WHERE c.tier = 'premium';

-- Update with aggregated data
UPDATE products p
SET avg_rating = (
    SELECT AVG(rating)
    FROM reviews r
    WHERE r.product_id = p.id
);
```

### 3.4 UPDATE with RETURNING

```sql
-- PostgreSQL: Return updated rows
UPDATE users
SET status = 'verified'
WHERE email_verified = true AND status = 'pending'
RETURNING id, username, email;

-- Use in CTE
WITH updated AS (
    UPDATE orders
    SET status = 'shipped'
    WHERE status = 'processing'
    RETURNING id, customer_id
)
INSERT INTO notifications (user_id, message)
SELECT customer_id, 'Your order has shipped!'
FROM updated;
```

### 3.5 Safe UPDATE Patterns

```sql
-- Always use WHERE clause
-- NEVER: UPDATE users SET status = 'inactive';

-- Use transactions for critical updates
BEGIN;
    SELECT * FROM orders WHERE id = 1 FOR UPDATE;  -- Lock row
    UPDATE orders SET status = 'processing' WHERE id = 1;
COMMIT;

-- Limit updates (MySQL)
UPDATE users SET status = 'inactive'
WHERE last_login < '2023-01-01'
LIMIT 1000;

-- Update with row count check
DO $$
DECLARE
    rows_affected INTEGER;
BEGIN
    UPDATE orders SET status = 'cancelled' WHERE id = 999;
    GET DIAGNOSTICS rows_affected = ROW_COUNT;
    IF rows_affected = 0 THEN
        RAISE EXCEPTION 'Order not found';
    END IF;
END $$;
```

---

## 4. DELETE Statements

### 4.1 Basic DELETE

```sql
-- Delete specific rows
DELETE FROM users WHERE id = 1;

-- Delete with multiple conditions
DELETE FROM sessions
WHERE user_id = 1 AND expires_at < NOW();

-- Delete all rows (use TRUNCATE for better performance)
DELETE FROM temp_logs;
```

### 4.2 DELETE with Subquery

```sql
-- Delete based on another table
DELETE FROM orders
WHERE customer_id IN (
    SELECT id FROM customers WHERE status = 'deleted'
);

-- Delete with NOT EXISTS
DELETE FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.id
);
```

### 4.3 DELETE with JOIN

```sql
-- PostgreSQL: DELETE USING
DELETE FROM order_items oi
USING orders o
WHERE oi.order_id = o.id
  AND o.status = 'cancelled';

-- MySQL: DELETE JOIN
DELETE oi
FROM order_items oi
JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'cancelled';

-- Delete from multiple tables (MySQL)
DELETE o, oi
FROM orders o
JOIN order_items oi ON o.id = oi.order_id
WHERE o.status = 'cancelled';
```

### 4.4 DELETE with RETURNING

```sql
-- PostgreSQL: Return deleted rows
DELETE FROM expired_sessions
WHERE expires_at < NOW()
RETURNING id, user_id;

-- Archive before delete
WITH deleted AS (
    DELETE FROM orders
    WHERE created_at < '2020-01-01'
    RETURNING *
)
INSERT INTO orders_archive
SELECT * FROM deleted;
```

### 4.5 Soft Delete Pattern

```sql
-- Instead of DELETE, use soft delete
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP;

-- "Delete" by setting timestamp
UPDATE users SET deleted_at = NOW() WHERE id = 1;

-- Query only active records
SELECT * FROM users WHERE deleted_at IS NULL;

-- Create view for convenience
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;

-- Permanently delete after retention period
DELETE FROM users
WHERE deleted_at IS NOT NULL
  AND deleted_at < NOW() - INTERVAL '30 days';
```

---

## 5. MERGE (UPSERT)

### 5.1 SQL Standard MERGE

```sql
-- Standard SQL MERGE (PostgreSQL 15+, SQL Server, Oracle)
MERGE INTO inventory AS target
USING (
    SELECT product_id, quantity
    FROM incoming_shipment
) AS source
ON target.product_id = source.product_id
WHEN MATCHED THEN
    UPDATE SET quantity = target.quantity + source.quantity
WHEN NOT MATCHED THEN
    INSERT (product_id, quantity)
    VALUES (source.product_id, source.quantity);

-- With DELETE action
MERGE INTO products AS target
USING product_updates AS source
ON target.id = source.id
WHEN MATCHED AND source.deleted = true THEN
    DELETE
WHEN MATCHED THEN
    UPDATE SET
        name = source.name,
        price = source.price
WHEN NOT MATCHED THEN
    INSERT (name, price)
    VALUES (source.name, source.price);
```

### 5.2 Database-Specific UPSERT

```sql
-- PostgreSQL: INSERT ... ON CONFLICT
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99)
ON CONFLICT (sku) DO UPDATE SET
    name = EXCLUDED.name,
    price = EXCLUDED.price,
    updated_at = NOW();

-- MySQL: INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    price = VALUES(price),
    updated_at = NOW();

-- MySQL: REPLACE INTO (deletes then inserts)
REPLACE INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99);

-- SQLite: INSERT OR REPLACE
INSERT OR REPLACE INTO products (sku, name, price)
VALUES ('ABC123', 'Widget', 29.99);
```

---

## 6. Bulk Operations

### 6.1 Efficient Bulk Insert

```sql
-- Use multi-value INSERT
INSERT INTO logs (level, message, created_at) VALUES
    ('INFO', 'Message 1', NOW()),
    ('WARN', 'Message 2', NOW()),
    ('ERROR', 'Message 3', NOW());
    -- Can insert thousands of rows

-- PostgreSQL: COPY (fastest for large datasets)
COPY users (username, email, created_at)
FROM '/path/to/users.csv'
WITH (FORMAT CSV, HEADER true);

-- MySQL: LOAD DATA
LOAD DATA INFILE '/path/to/users.csv'
INTO TABLE users
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### 6.2 Batch Updates

```sql
-- Update in batches to avoid long locks
DO $$
DECLARE
    batch_size INTEGER := 1000;
    rows_updated INTEGER;
BEGIN
    LOOP
        UPDATE orders
        SET status = 'archived'
        WHERE id IN (
            SELECT id FROM orders
            WHERE status = 'old' AND archived_at IS NULL
            LIMIT batch_size
            FOR UPDATE SKIP LOCKED
        );

        GET DIAGNOSTICS rows_updated = ROW_COUNT;
        EXIT WHEN rows_updated = 0;

        COMMIT;
        PERFORM pg_sleep(0.1);  -- Brief pause
    END LOOP;
END $$;
```

---

## 7. Code Examples Across Languages

### Python (with psycopg2)

```python
import psycopg2
from psycopg2.extras import execute_batch, execute_values

conn = psycopg2.connect(database="mydb")

# Single insert
with conn.cursor() as cur:
    cur.execute(
        "INSERT INTO users (username, email) VALUES (%s, %s) RETURNING id",
        ("johndoe", "john@example.com")
    )
    user_id = cur.fetchone()[0]
    conn.commit()

# Bulk insert with execute_batch (efficient)
users = [
    ("user1", "user1@example.com"),
    ("user2", "user2@example.com"),
    ("user3", "user3@example.com"),
]
with conn.cursor() as cur:
    execute_batch(
        cur,
        "INSERT INTO users (username, email) VALUES (%s, %s)",
        users,
        page_size=100
    )
    conn.commit()

# Even faster: execute_values
with conn.cursor() as cur:
    execute_values(
        cur,
        "INSERT INTO users (username, email) VALUES %s",
        users,
        template="(%s, %s)"
    )
    conn.commit()

# Update with transaction
try:
    with conn.cursor() as cur:
        cur.execute(
            "UPDATE accounts SET balance = balance - %s WHERE id = %s",
            (100, 1)
        )
        cur.execute(
            "UPDATE accounts SET balance = balance + %s WHERE id = %s",
            (100, 2)
        )
        conn.commit()
except Exception as e:
    conn.rollback()
    raise
```

### Java (JDBC with Batch)

```java
import java.sql.*;
import java.util.List;

public class DMLExample {

    public int insertUser(Connection conn, String username, String email)
            throws SQLException {
        String sql = "INSERT INTO users (username, email) VALUES (?, ?) RETURNING id";
        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, username);
            stmt.setString(2, email);
            try (ResultSet rs = stmt.executeQuery()) {
                if (rs.next()) {
                    return rs.getInt("id");
                }
            }
        }
        return -1;
    }

    public void batchInsert(Connection conn, List<User> users) throws SQLException {
        String sql = "INSERT INTO users (username, email) VALUES (?, ?)";
        conn.setAutoCommit(false);

        try (PreparedStatement stmt = conn.prepareStatement(sql)) {
            for (User user : users) {
                stmt.setString(1, user.getUsername());
                stmt.setString(2, user.getEmail());
                stmt.addBatch();
            }
            stmt.executeBatch();
            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }

    public void transferFunds(Connection conn, int from, int to, double amount)
            throws SQLException {
        conn.setAutoCommit(false);

        String debit = "UPDATE accounts SET balance = balance - ? WHERE id = ?";
        String credit = "UPDATE accounts SET balance = balance + ? WHERE id = ?";

        try (PreparedStatement debitStmt = conn.prepareStatement(debit);
             PreparedStatement creditStmt = conn.prepareStatement(credit)) {

            debitStmt.setDouble(1, amount);
            debitStmt.setInt(2, from);
            debitStmt.executeUpdate();

            creditStmt.setDouble(1, amount);
            creditStmt.setInt(2, to);
            creditStmt.executeUpdate();

            conn.commit();
        } catch (SQLException e) {
            conn.rollback();
            throw e;
        } finally {
            conn.setAutoCommit(true);
        }
    }
}
```

### JavaScript (Knex.js)

```javascript
const knex = require('knex')(config);

// Insert single row
const [userId] = await knex('users')
    .insert({ username: 'johndoe', email: 'john@example.com' })
    .returning('id');

// Bulk insert
await knex('users').insert([
    { username: 'user1', email: 'user1@example.com' },
    { username: 'user2', email: 'user2@example.com' },
    { username: 'user3', email: 'user3@example.com' },
]);

// Upsert (PostgreSQL)
await knex('users')
    .insert({ email: 'john@example.com', username: 'johndoe', login_count: 1 })
    .onConflict('email')
    .merge({ login_count: knex.raw('users.login_count + 1') });

// Update with join
await knex('orders')
    .join('customers', 'orders.customer_id', 'customers.id')
    .where('customers.tier', 'premium')
    .update({ status: 'priority' });

// Transaction
await knex.transaction(async (trx) => {
    await trx('accounts')
        .where('id', 1)
        .decrement('balance', 100);

    await trx('accounts')
        .where('id', 2)
        .increment('balance', 100);
});

// Batch update with chunks
const ids = [1, 2, 3, /* ... thousands of ids */];
const chunks = _.chunk(ids, 1000);

for (const chunk of chunks) {
    await knex('users')
        .whereIn('id', chunk)
        .update({ status: 'processed' });
}
```

---

## 8. Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DML BEST PRACTICES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SAFETY                                                                     │
│   ──────                                                                     │
│   • ALWAYS use WHERE clause with UPDATE and DELETE                          │
│   • Use transactions for multi-statement operations                         │
│   • Test queries with SELECT first before UPDATE/DELETE                     │
│   • Use LIMIT when updating large datasets (batch processing)               │
│   • Implement soft deletes for important data                               │
│                                                                              │
│   PERFORMANCE                                                                │
│   ───────────                                                                │
│   • Use multi-value INSERT for bulk operations                              │
│   • Use COPY/LOAD DATA for very large imports                               │
│   • Batch updates to avoid long-running transactions                        │
│   • Use RETURNING to avoid extra SELECT queries                             │
│   • Consider disabling indexes during bulk loads                            │
│                                                                              │
│   SECURITY                                                                   │
│   ────────                                                                   │
│   • ALWAYS use parameterized queries (prevent SQL injection)                │
│   • Never concatenate user input into SQL strings                           │
│   • Validate and sanitize input at application layer                        │
│   • Use least-privilege database accounts                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

| Command | Purpose | Affects | Reversible |
|---------|---------|---------|------------|
| INSERT | Add new rows | Data | Yes (in transaction) |
| UPDATE | Modify existing rows | Data | Yes (in transaction) |
| DELETE | Remove rows | Data | Yes (in transaction) |
| MERGE | Upsert operations | Data | Yes (in transaction) |

DML operations are the core of database interaction. Understanding efficient patterns for bulk operations, proper transaction handling, and security best practices is essential for production applications.
