# DDL - Data Definition Language

## 1. Introduction

**Data Definition Language (DDL)** consists of SQL commands used to define, modify, and delete database structures. DDL statements affect the schema (structure) rather than the data itself.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DDL COMMANDS                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   CREATE    - Create new database objects (tables, indexes, views)          │
│   ALTER     - Modify existing database objects                              │
│   DROP      - Delete database objects                                       │
│   TRUNCATE  - Remove all data from a table (keeps structure)               │
│   RENAME    - Rename database objects                                       │
│   COMMENT   - Add comments to data dictionary                               │
│                                                                              │
│   DDL statements are auto-committed in most databases                       │
│   (cannot be rolled back)                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. CREATE Statements

### 2.1 CREATE DATABASE

```sql
-- Create a new database
CREATE DATABASE ecommerce;

-- With options (PostgreSQL)
CREATE DATABASE ecommerce
    WITH
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0;

-- MySQL with character set
CREATE DATABASE ecommerce
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

-- Create if not exists
CREATE DATABASE IF NOT EXISTS ecommerce;
```

### 2.2 CREATE TABLE

```sql
-- Basic table creation
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- With all constraint types
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    customer_id INTEGER NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL CHECK (total_amount >= 0),
    status VARCHAR(20) DEFAULT 'pending'
        CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    shipping_address TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    -- Foreign key constraint
    CONSTRAINT fk_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Create table from another table
CREATE TABLE orders_archive AS
SELECT * FROM orders WHERE created_at < '2023-01-01';

-- Create table with same structure (no data)
CREATE TABLE orders_backup (LIKE orders INCLUDING ALL);
```

### 2.3 Data Types

```sql
-- Numeric types
CREATE TABLE numeric_examples (
    -- Integers
    tiny_int SMALLINT,              -- -32,768 to 32,767
    regular_int INTEGER,            -- -2.1B to 2.1B
    big_int BIGINT,                 -- Very large integers

    -- Exact decimals (for money)
    price DECIMAL(10, 2),           -- 10 digits, 2 after decimal
    tax_rate NUMERIC(5, 4),         -- 5 digits, 4 after decimal

    -- Floating point (approximate)
    measurement REAL,               -- 6 decimal digits precision
    scientific DOUBLE PRECISION,    -- 15 decimal digits precision

    -- Auto-increment
    id SERIAL,                      -- PostgreSQL
    -- id INT AUTO_INCREMENT,       -- MySQL
    -- id INTEGER PRIMARY KEY,      -- SQLite (auto-increments)

    -- Boolean
    is_active BOOLEAN DEFAULT true
);

-- String types
CREATE TABLE string_examples (
    -- Fixed length (padded with spaces)
    country_code CHAR(2),           -- Always 2 characters

    -- Variable length
    name VARCHAR(100),              -- Up to 100 characters
    description TEXT,               -- Unlimited length

    -- Binary data
    file_data BYTEA,                -- PostgreSQL
    -- file_data BLOB,              -- MySQL

    -- UUID
    guid UUID DEFAULT gen_random_uuid()  -- PostgreSQL
);

-- Date and Time types
CREATE TABLE datetime_examples (
    -- Date only
    birth_date DATE,

    -- Time only
    start_time TIME,
    start_time_tz TIME WITH TIME ZONE,

    -- Date and time
    created_at TIMESTAMP,
    created_at_tz TIMESTAMPTZ,      -- With timezone (recommended)

    -- Interval
    duration INTERVAL
);

-- JSON types (PostgreSQL)
CREATE TABLE json_examples (
    data JSON,                      -- Stored as text
    data_binary JSONB               -- Binary, indexable, faster queries
);

-- Array types (PostgreSQL)
CREATE TABLE array_examples (
    tags TEXT[],
    scores INTEGER[],
    matrix INTEGER[][]
);

-- Enum types
CREATE TYPE status_enum AS ENUM ('pending', 'active', 'inactive');
CREATE TABLE enum_example (
    status status_enum DEFAULT 'pending'
);
```

### 2.4 CREATE INDEX

```sql
-- Basic index
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Composite index
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at DESC);

-- Partial index (PostgreSQL)
CREATE INDEX idx_orders_pending ON orders(created_at)
WHERE status = 'pending';

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- Covering index (includes additional columns)
CREATE INDEX idx_orders_covering ON orders(customer_id)
INCLUDE (total_amount, status);

-- Concurrent index creation (no table lock)
CREATE INDEX CONCURRENTLY idx_large_table ON large_table(column);

-- Full-text search index (PostgreSQL)
CREATE INDEX idx_products_search ON products
USING GIN (to_tsvector('english', name || ' ' || description));

-- GiST index for geometric/range data
CREATE INDEX idx_locations_geo ON locations USING GIST (coordinates);
```

### 2.5 CREATE VIEW

```sql
-- Basic view
CREATE VIEW active_customers AS
SELECT id, name, email
FROM customers
WHERE status = 'active';

-- View with joins
CREATE VIEW order_details AS
SELECT
    o.id AS order_id,
    o.order_number,
    c.name AS customer_name,
    c.email AS customer_email,
    o.total_amount,
    o.status,
    o.created_at
FROM orders o
JOIN customers c ON o.customer_id = c.id;

-- Create or replace
CREATE OR REPLACE VIEW order_summary AS
SELECT
    customer_id,
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_spent,
    AVG(total_amount) AS avg_order_value
FROM orders
GROUP BY customer_id;

-- Materialized view (PostgreSQL) - stores results
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT
    DATE_TRUNC('month', created_at) AS month,
    SUM(total_amount) AS total_sales,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC('month', created_at)
WITH DATA;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW monthly_sales;
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales;  -- No lock
```

---

## 3. ALTER Statements

### 3.1 ALTER TABLE

```sql
-- Add column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users ADD COLUMN bio TEXT DEFAULT '';

-- Add multiple columns
ALTER TABLE users
    ADD COLUMN first_name VARCHAR(50),
    ADD COLUMN last_name VARCHAR(50);

-- Drop column
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users DROP COLUMN IF EXISTS phone;

-- Rename column
ALTER TABLE users RENAME COLUMN username TO user_name;

-- Change column type
ALTER TABLE users ALTER COLUMN bio TYPE VARCHAR(500);

-- With conversion
ALTER TABLE users
ALTER COLUMN price TYPE INTEGER
USING price::INTEGER;

-- Set/drop default
ALTER TABLE users ALTER COLUMN status SET DEFAULT 'active';
ALTER TABLE users ALTER COLUMN status DROP DEFAULT;

-- Set/drop NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone DROP NOT NULL;

-- Add constraint
ALTER TABLE orders
ADD CONSTRAINT chk_positive_amount
CHECK (total_amount > 0);

ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(id);

-- Drop constraint
ALTER TABLE orders DROP CONSTRAINT chk_positive_amount;

-- Rename table
ALTER TABLE users RENAME TO app_users;

-- Change table owner (PostgreSQL)
ALTER TABLE users OWNER TO new_owner;
```

### 3.2 ALTER INDEX

```sql
-- Rename index
ALTER INDEX idx_users_email RENAME TO idx_users_email_address;

-- Rebuild index (PostgreSQL)
REINDEX INDEX idx_users_email;
REINDEX TABLE users;

-- Set index options
ALTER INDEX idx_users_email SET (fillfactor = 80);
```

### 3.3 ALTER DATABASE

```sql
-- Rename database
ALTER DATABASE old_name RENAME TO new_name;

-- Change owner
ALTER DATABASE ecommerce OWNER TO new_owner;

-- Set connection limit
ALTER DATABASE ecommerce CONNECTION LIMIT 100;

-- Set configuration
ALTER DATABASE ecommerce SET timezone TO 'UTC';
```

---

## 4. DROP Statements

```sql
-- Drop table
DROP TABLE users;
DROP TABLE IF EXISTS users;

-- Drop with cascade (removes dependent objects)
DROP TABLE customers CASCADE;

-- Drop multiple tables
DROP TABLE IF EXISTS orders, order_items, customers CASCADE;

-- Drop database
DROP DATABASE ecommerce;
DROP DATABASE IF EXISTS ecommerce;

-- Drop index
DROP INDEX idx_users_email;
DROP INDEX CONCURRENTLY idx_users_email;  -- No lock

-- Drop view
DROP VIEW active_customers;
DROP VIEW IF EXISTS active_customers CASCADE;

-- Drop materialized view
DROP MATERIALIZED VIEW monthly_sales;

-- Drop constraint
ALTER TABLE orders DROP CONSTRAINT fk_customer;

-- Drop type
DROP TYPE status_enum;
DROP TYPE IF EXISTS status_enum CASCADE;
```

---

## 5. TRUNCATE

```sql
-- Remove all rows (faster than DELETE)
TRUNCATE TABLE logs;

-- Reset identity/serial counter
TRUNCATE TABLE users RESTART IDENTITY;

-- Truncate with cascade (truncate referencing tables)
TRUNCATE TABLE customers CASCADE;

-- Truncate multiple tables
TRUNCATE TABLE orders, order_items, payments;

-- Note: TRUNCATE cannot be rolled back in most databases
-- (MySQL InnoDB is an exception - it's transactional)
```

---

## 6. Schema Management

```sql
-- Create schema
CREATE SCHEMA sales;
CREATE SCHEMA IF NOT EXISTS inventory;

-- Create table in schema
CREATE TABLE sales.orders (
    id SERIAL PRIMARY KEY,
    amount DECIMAL(10, 2)
);

-- Set search path (PostgreSQL)
SET search_path TO sales, public;

-- Drop schema
DROP SCHEMA sales;
DROP SCHEMA sales CASCADE;  -- Drop all objects in schema

-- Move table to different schema
ALTER TABLE orders SET SCHEMA archive;
```

---

## 7. Code Examples Across Languages

### Python (SQLAlchemy)

```python
from sqlalchemy import create_engine, MetaData, Table, Column
from sqlalchemy import Integer, String, DateTime, Numeric, Boolean, ForeignKey
from sqlalchemy.sql import func

engine = create_engine('postgresql://user:pass@localhost/db')
metadata = MetaData()

# Define table
users = Table('users', metadata,
    Column('id', Integer, primary_key=True),
    Column('username', String(50), unique=True, nullable=False),
    Column('email', String(255), unique=True, nullable=False),
    Column('is_active', Boolean, default=True),
    Column('created_at', DateTime, server_default=func.now())
)

orders = Table('orders', metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', Integer, ForeignKey('users.id', ondelete='CASCADE')),
    Column('total', Numeric(10, 2), nullable=False),
    Column('created_at', DateTime, server_default=func.now())
)

# Create tables
metadata.create_all(engine)

# Drop tables
metadata.drop_all(engine)

# Alter table (using raw SQL)
with engine.connect() as conn:
    conn.execute("ALTER TABLE users ADD COLUMN phone VARCHAR(20)")
```

### Java (JDBC)

```java
import java.sql.*;

public class DDLExample {
    public static void main(String[] args) {
        String url = "jdbc:postgresql://localhost/db";

        try (Connection conn = DriverManager.getConnection(url, "user", "pass");
             Statement stmt = conn.createStatement()) {

            // Create table
            String createTable = """
                CREATE TABLE IF NOT EXISTS users (
                    id SERIAL PRIMARY KEY,
                    username VARCHAR(50) UNIQUE NOT NULL,
                    email VARCHAR(255) UNIQUE NOT NULL,
                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """;
            stmt.execute(createTable);

            // Create index
            stmt.execute("CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)");

            // Alter table
            stmt.execute("ALTER TABLE users ADD COLUMN IF NOT EXISTS phone VARCHAR(20)");

            // Drop table
            stmt.execute("DROP TABLE IF EXISTS temp_table");

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### JavaScript (Knex.js Migrations)

```javascript
// migrations/20240115_create_users.js
exports.up = function(knex) {
    return knex.schema
        .createTable('users', table => {
            table.increments('id').primary();
            table.string('username', 50).unique().notNullable();
            table.string('email', 255).unique().notNullable();
            table.boolean('is_active').defaultTo(true);
            table.timestamps(true, true); // created_at, updated_at
        })
        .createTable('orders', table => {
            table.increments('id').primary();
            table.integer('user_id').unsigned()
                .references('id').inTable('users')
                .onDelete('CASCADE');
            table.decimal('total', 10, 2).notNullable();
            table.enum('status', ['pending', 'completed', 'cancelled'])
                .defaultTo('pending');
            table.timestamps(true, true);

            // Indexes
            table.index(['user_id', 'created_at']);
        });
};

exports.down = function(knex) {
    return knex.schema
        .dropTableIfExists('orders')
        .dropTableIfExists('users');
};

// Alter table migration
exports.up = function(knex) {
    return knex.schema.alterTable('users', table => {
        table.string('phone', 20);
        table.text('bio');
    });
};

exports.down = function(knex) {
    return knex.schema.alterTable('users', table => {
        table.dropColumn('phone');
        table.dropColumn('bio');
    });
};
```

---

## 8. Best Practices

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DDL BEST PRACTICES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   NAMING CONVENTIONS                                                         │
│   ──────────────────                                                         │
│   • Tables: lowercase, plural (users, orders, order_items)                  │
│   • Columns: lowercase, snake_case (created_at, user_id)                    │
│   • Indexes: idx_tablename_columns (idx_users_email)                        │
│   • Foreign keys: fk_tablename_reference (fk_orders_customer)               │
│   • Constraints: chk_tablename_description (chk_orders_positive_amount)     │
│                                                                              │
│   SAFETY MEASURES                                                            │
│   ───────────────                                                            │
│   • Always use IF EXISTS / IF NOT EXISTS                                    │
│   • Test DDL in non-production first                                        │
│   • Use transactions where supported                                        │
│   • Create indexes CONCURRENTLY on large tables                             │
│   • Back up before destructive operations                                   │
│                                                                              │
│   PERFORMANCE                                                                │
│   ───────────                                                                │
│   • Add indexes for frequently queried columns                              │
│   • Don't over-index (slows writes)                                        │
│   • Use appropriate data types (don't use TEXT for short strings)          │
│   • Consider partitioning for very large tables                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

| Command | Purpose | Can Rollback? |
|---------|---------|---------------|
| CREATE | Create objects | No* |
| ALTER | Modify objects | No* |
| DROP | Delete objects | No* |
| TRUNCATE | Remove all data | No* |

*DDL statements are auto-committed in most databases. PostgreSQL allows DDL within transactions.

DDL forms the foundation of database structure. Proper schema design with appropriate constraints and indexes is crucial for data integrity and performance.
