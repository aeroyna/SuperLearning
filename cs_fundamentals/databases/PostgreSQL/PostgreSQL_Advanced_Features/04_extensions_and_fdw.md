# Extensions and Foreign Data Wrappers

## Learning Objectives
- Understand PostgreSQL's extension architecture
- Install and configure popular extensions
- Use Foreign Data Wrappers to access external data
- Create custom extensions

---

## 1. Extension Architecture

### What are Extensions?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Extension System                       │
│                                                                      │
│  Extension = Packaged database objects                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Functions (SQL, PL/pgSQL, C)                             │    │
│  │  • Data types                                                │    │
│  │  • Operators                                                 │    │
│  │  • Index access methods                                      │    │
│  │  • Tables and views                                          │    │
│  │  • Configuration settings                                    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Benefits:                                                           │
│  • Easy installation (CREATE EXTENSION)                              │
│  • Version management and upgrades                                   │
│  • Dependency tracking                                               │
│  • Clean uninstallation                                              │
│  • Modular functionality                                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Extension Management

```sql
-- List available extensions
SELECT * FROM pg_available_extensions ORDER BY name;

-- List installed extensions
SELECT * FROM pg_extension;

-- Install extension
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Install specific version
CREATE EXTENSION hstore VERSION '1.4';

-- Install in specific schema
CREATE EXTENSION ltree SCHEMA public;

-- Upgrade extension
ALTER EXTENSION pg_trgm UPDATE TO '1.6';

-- Remove extension
DROP EXTENSION pg_trgm;

-- Show extension objects
SELECT * FROM pg_extension_objects('pg_trgm');
-- Or
\dx+ pg_trgm
```

---

## 2. Essential Extensions

### pg_stat_statements

```sql
-- Track query statistics
-- Must add to shared_preload_libraries in postgresql.conf

-- Install
CREATE EXTENSION pg_stat_statements;

-- View query statistics
SELECT
    substring(query, 1, 50) AS query,
    calls,
    total_exec_time / 1000 AS total_seconds,
    mean_exec_time AS avg_ms,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Find slow queries
SELECT
    query,
    calls,
    mean_exec_time AS avg_ms,
    stddev_exec_time AS stddev_ms
FROM pg_stat_statements
WHERE calls > 100
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Reset statistics
SELECT pg_stat_statements_reset();
```

### pg_trgm (Trigram Similarity)

```sql
-- Fuzzy string matching
CREATE EXTENSION pg_trgm;

-- Similarity functions
SELECT similarity('word', 'wrod');           -- 0.5
SELECT word_similarity('word', 'wording');   -- 1.0

-- Similarity operators
SELECT 'word' % 'wrod';                      -- true (default threshold 0.3)
SELECT 'word' <% 'wording';                  -- true (word similarity)

-- Trigram index for LIKE/ILIKE
CREATE INDEX idx_name_trgm ON users USING GIN (name gin_trgm_ops);

-- Fast pattern matching
SELECT * FROM users WHERE name ILIKE '%john%';  -- Uses index
SELECT * FROM users WHERE name % 'johnn';        -- Fuzzy match

-- Autocomplete
SELECT name FROM users
WHERE name % 'joh'
ORDER BY similarity(name, 'joh') DESC
LIMIT 10;
```

### uuid-ossp

```sql
-- UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Generate UUIDs
SELECT uuid_generate_v4();  -- Random UUID
SELECT uuid_generate_v1();  -- Time-based UUID

-- Use in table
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    customer_name VARCHAR(100)
);

-- Or use built-in (PostgreSQL 13+)
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100)
);
```

### pgcrypto

```sql
-- Cryptographic functions
CREATE EXTENSION pgcrypto;

-- Password hashing
INSERT INTO users (email, password_hash) VALUES
    ('user@example.com', crypt('mypassword', gen_salt('bf')));

-- Verify password
SELECT * FROM users
WHERE email = 'user@example.com'
  AND password_hash = crypt('mypassword', password_hash);

-- Generate random bytes
SELECT encode(gen_random_bytes(16), 'hex');

-- Encryption
SELECT pgp_sym_encrypt('secret data', 'encryption_key');
SELECT pgp_sym_decrypt(
    pgp_sym_encrypt('secret data', 'key'),
    'key'
);

-- Hashing
SELECT digest('data', 'sha256');
SELECT encode(digest('data', 'sha256'), 'hex');
```

### hstore

```sql
-- Key-value store
CREATE EXTENSION hstore;

-- Create table with hstore
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes HSTORE
);

INSERT INTO products (name, attributes) VALUES
    ('Laptop', 'brand => Apple, cpu => M1, ram => 16GB'),
    ('Phone', 'brand => Samsung, screen => 6.5, 5g => true');

-- Query hstore
SELECT name, attributes -> 'brand' AS brand FROM products;
SELECT * FROM products WHERE attributes ? 'cpu';
SELECT * FROM products WHERE attributes @> 'brand => Apple';

-- Update hstore
UPDATE products SET attributes = attributes || 'color => silver'
WHERE name = 'Laptop';

-- Delete key
UPDATE products SET attributes = delete(attributes, 'color')
WHERE name = 'Laptop';

-- Convert to JSONB (modern alternative)
SELECT attributes::jsonb FROM products;
```

### ltree

```sql
-- Hierarchical data
CREATE EXTENSION ltree;

-- Categories tree
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    path LTREE
);

INSERT INTO categories (name, path) VALUES
    ('Electronics', 'electronics'),
    ('Computers', 'electronics.computers'),
    ('Laptops', 'electronics.computers.laptops'),
    ('Phones', 'electronics.phones'),
    ('Android', 'electronics.phones.android'),
    ('iOS', 'electronics.phones.ios');

CREATE INDEX idx_categories_path ON categories USING GIST (path);

-- Find descendants
SELECT * FROM categories WHERE path <@ 'electronics.phones';

-- Find ancestors
SELECT * FROM categories WHERE path @> 'electronics.computers.laptops';

-- Pattern matching
SELECT * FROM categories WHERE path ~ 'electronics.*.laptops';

-- Get path level
SELECT name, nlevel(path) FROM categories;
```

---

## 3. Specialized Extensions

### PostGIS (Geospatial)

```sql
-- Geographic data support
CREATE EXTENSION postgis;

-- Store locations
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    location GEOGRAPHY(POINT, 4326)  -- WGS84
);

INSERT INTO locations (name, location) VALUES
    ('New York', ST_GeographyFromText('POINT(-74.006 40.7128)')),
    ('Los Angeles', ST_GeographyFromText('POINT(-118.2437 34.0522)')),
    ('Chicago', ST_GeographyFromText('POINT(-87.6298 41.8781)'));

-- Distance calculation
SELECT
    a.name AS from_city,
    b.name AS to_city,
    ST_Distance(a.location, b.location) / 1000 AS distance_km
FROM locations a, locations b
WHERE a.name = 'New York';

-- Find nearby points
CREATE INDEX idx_locations_geo ON locations USING GIST (location);

SELECT name
FROM locations
WHERE ST_DWithin(
    location,
    ST_GeographyFromText('POINT(-74.006 40.7128)'),
    500000  -- 500km
);
```

### TimescaleDB (Time-Series)

```sql
-- Time-series optimization
CREATE EXTENSION timescaledb;

-- Create hypertable
CREATE TABLE metrics (
    time TIMESTAMPTZ NOT NULL,
    device_id INTEGER,
    temperature DOUBLE PRECISION,
    humidity DOUBLE PRECISION
);

SELECT create_hypertable('metrics', 'time');

-- Insert data
INSERT INTO metrics VALUES
    (NOW(), 1, 22.5, 45.0),
    (NOW() - INTERVAL '1 hour', 1, 21.8, 48.0);

-- Time-series queries
SELECT time_bucket('1 hour', time) AS hour,
       device_id,
       AVG(temperature) AS avg_temp
FROM metrics
WHERE time > NOW() - INTERVAL '1 day'
GROUP BY hour, device_id
ORDER BY hour;

-- Continuous aggregates
CREATE MATERIALIZED VIEW hourly_metrics
WITH (timescaledb.continuous) AS
SELECT time_bucket('1 hour', time) AS bucket,
       device_id,
       AVG(temperature) AS avg_temp,
       MAX(temperature) AS max_temp
FROM metrics
GROUP BY bucket, device_id;
```

### pgvector (Vector Similarity)

```sql
-- Vector similarity search (AI embeddings)
CREATE EXTENSION vector;

-- Store embeddings
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536)  -- OpenAI embedding dimension
);

-- Create index
CREATE INDEX idx_documents_embedding ON documents
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- Similarity search
SELECT content, 1 - (embedding <=> '[0.1, 0.2, ...]'::vector) AS similarity
FROM documents
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 10;

-- Operators:
-- <-> L2 distance
-- <=> Cosine distance
-- <#> Inner product (negative)
```

---

## 4. Foreign Data Wrappers

### FDW Concept

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Foreign Data Wrapper Architecture                 │
│                                                                      │
│  PostgreSQL                        External Data Sources            │
│  ┌─────────────┐                   ┌─────────────────────┐          │
│  │   Query     │                   │  PostgreSQL Server  │          │
│  └──────┬──────┘                   │  MySQL Server       │          │
│         │                          │  MongoDB            │          │
│         ▼                          │  CSV Files          │          │
│  ┌─────────────┐                   │  REST APIs          │          │
│  │    FDW      │◄─────────────────►│  S3 Buckets         │          │
│  │   Layer     │                   │  Redis              │          │
│  └─────────────┘                   └─────────────────────┘          │
│         │                                                            │
│         ▼                                                            │
│  ┌─────────────┐                                                     │
│  │  Foreign    │  Looks like a regular table                        │
│  │   Table     │  SELECT, INSERT, UPDATE, DELETE                    │
│  └─────────────┘                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### postgres_fdw (Connect to PostgreSQL)

```sql
-- Connect to another PostgreSQL database
CREATE EXTENSION postgres_fdw;

-- Create server connection
CREATE SERVER remote_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote.example.com', port '5432', dbname 'remote_db');

-- Create user mapping
CREATE USER MAPPING FOR local_user
SERVER remote_server
OPTIONS (user 'remote_user', password 'remote_password');

-- Import foreign schema
IMPORT FOREIGN SCHEMA public
FROM SERVER remote_server
INTO local_schema;

-- Or create individual foreign table
CREATE FOREIGN TABLE remote_users (
    id INTEGER,
    name VARCHAR(100),
    email VARCHAR(255)
)
SERVER remote_server
OPTIONS (schema_name 'public', table_name 'users');

-- Query foreign table like local
SELECT * FROM remote_users WHERE id = 1;

-- Join local and remote tables
SELECT o.id, o.total, u.name
FROM orders o
JOIN remote_users u ON o.user_id = u.id;
```

### file_fdw (Read Files)

```sql
-- Read CSV and other files
CREATE EXTENSION file_fdw;

CREATE SERVER file_server FOREIGN DATA WRAPPER file_fdw;

-- Create foreign table for CSV
CREATE FOREIGN TABLE log_data (
    timestamp TIMESTAMP,
    level VARCHAR(10),
    message TEXT
)
SERVER file_server
OPTIONS (
    filename '/var/log/app/application.csv',
    format 'csv',
    header 'true'
);

-- Query file like table
SELECT * FROM log_data
WHERE level = 'ERROR'
  AND timestamp > NOW() - INTERVAL '1 day';

-- Program output
CREATE FOREIGN TABLE disk_usage (
    filesystem TEXT,
    size TEXT,
    used TEXT,
    available TEXT,
    use_percent TEXT,
    mount TEXT
)
SERVER file_server
OPTIONS (
    program 'df -h',
    format 'text',
    delimiter ' '
);
```

### mysql_fdw (Connect to MySQL)

```sql
-- Connect to MySQL (community extension)
CREATE EXTENSION mysql_fdw;

CREATE SERVER mysql_server
FOREIGN DATA WRAPPER mysql_fdw
OPTIONS (host 'mysql.example.com', port '3306');

CREATE USER MAPPING FOR postgres_user
SERVER mysql_server
OPTIONS (username 'mysql_user', password 'mysql_pass');

CREATE FOREIGN TABLE mysql_customers (
    id INTEGER,
    name VARCHAR(100),
    email VARCHAR(255)
)
SERVER mysql_server
OPTIONS (dbname 'shop', table_name 'customers');

-- Query MySQL from PostgreSQL
SELECT * FROM mysql_customers;
```

### mongo_fdw (Connect to MongoDB)

```sql
-- Connect to MongoDB (community extension)
CREATE EXTENSION mongo_fdw;

CREATE SERVER mongo_server
FOREIGN DATA WRAPPER mongo_fdw
OPTIONS (address 'mongo.example.com', port '27017');

CREATE USER MAPPING FOR postgres_user
SERVER mongo_server
OPTIONS (username 'mongo_user', password 'mongo_pass');

CREATE FOREIGN TABLE mongo_products (
    _id NAME,
    name VARCHAR(100),
    price NUMERIC,
    attributes JSONB
)
SERVER mongo_server
OPTIONS (database 'shop', collection 'products');

-- Query MongoDB documents
SELECT * FROM mongo_products WHERE price > 100;
```

---

## 5. FDW Optimization

### Push-Down Operations

```sql
-- postgres_fdw pushes down operations when possible
EXPLAIN VERBOSE
SELECT * FROM remote_users WHERE id = 1;
-- Remote SQL: SELECT id, name, email FROM public.users WHERE id = 1

-- Aggregate push-down (PostgreSQL 10+)
EXPLAIN VERBOSE
SELECT COUNT(*) FROM remote_users;
-- Remote SQL: SELECT count(*) FROM public.users
```

### FDW Options

```sql
-- Server options
ALTER SERVER remote_server OPTIONS (
    SET fetch_size '1000',      -- Rows per fetch
    SET extensions 'hstore'      -- Available extensions
);

-- Foreign table options
ALTER FOREIGN TABLE remote_users OPTIONS (
    SET updatable 'false'        -- Read-only
);

-- Cost estimation
ALTER FOREIGN TABLE remote_users OPTIONS (
    SET use_remote_estimate 'true'  -- Get row count from remote
);
```

---

## 6. Creating Custom Extensions

### Extension Structure

```
my_extension/
├── my_extension.control      # Metadata
├── my_extension--1.0.sql     # Installation script
├── my_extension--1.0--1.1.sql  # Upgrade script
├── Makefile                  # Build configuration
└── src/
    └── my_extension.c        # C source (optional)
```

### Control File

```ini
# my_extension.control
comment = 'My custom extension'
default_version = '1.0'
module_pathname = '$libdir/my_extension'
relocatable = true
requires = 'plpgsql'
```

### SQL Installation Script

```sql
-- my_extension--1.0.sql

-- Custom type
CREATE TYPE email_address AS (
    local_part VARCHAR(64),
    domain VARCHAR(255)
);

-- Custom function
CREATE OR REPLACE FUNCTION validate_email(email TEXT)
RETURNS BOOLEAN AS $$
BEGIN
    RETURN email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z]{2,}$';
END;
$$ LANGUAGE plpgsql IMMUTABLE;

-- Custom domain
CREATE DOMAIN valid_email AS TEXT
CHECK (validate_email(VALUE));

-- Custom operator
CREATE OPERATOR @@ (
    LEFTARG = TEXT,
    RIGHTARG = TEXT,
    FUNCTION = validate_email
);
```

### Upgrade Script

```sql
-- my_extension--1.0--1.1.sql

-- Add new function in version 1.1
CREATE OR REPLACE FUNCTION email_domain(email TEXT)
RETURNS TEXT AS $$
BEGIN
    RETURN split_part(email, '@', 2);
END;
$$ LANGUAGE plpgsql IMMUTABLE;
```

### Makefile

```makefile
EXTENSION = my_extension
DATA = my_extension--1.0.sql my_extension--1.0--1.1.sql

PG_CONFIG = pg_config
PGXS := $(shell $(PG_CONFIG) --pgxs)
include $(PGXS)
```

### Installation

```bash
# Build and install
make
make install

# In PostgreSQL
CREATE EXTENSION my_extension;
```

---

## 7. Extension Best Practices

### Version Management

```sql
-- Check installed version
SELECT extversion FROM pg_extension WHERE extname = 'pg_trgm';

-- Check available versions
SELECT * FROM pg_available_extension_versions
WHERE name = 'pg_trgm';

-- Upgrade path
ALTER EXTENSION pg_trgm UPDATE;  -- To latest
ALTER EXTENSION pg_trgm UPDATE TO '1.6';  -- To specific
```

### Schema Management

```sql
-- Install in specific schema
CREATE EXTENSION pg_trgm SCHEMA extensions;

-- Move extension objects
ALTER EXTENSION pg_trgm SET SCHEMA public;

-- Recommended: Use dedicated schema for extensions
CREATE SCHEMA IF NOT EXISTS extensions;
GRANT USAGE ON SCHEMA extensions TO PUBLIC;

SET search_path = public, extensions;
```

### Security

```sql
-- Extension requires superuser by default
-- Grant extension usage to specific roles
GRANT USAGE ON FOREIGN DATA WRAPPER postgres_fdw TO app_user;

-- Restrict foreign table access
REVOKE ALL ON remote_users FROM PUBLIC;
GRANT SELECT ON remote_users TO app_reader;
```

---

## 8. Popular Extensions Reference

| Extension | Purpose | Category |
|-----------|---------|----------|
| pg_stat_statements | Query statistics | Monitoring |
| pg_trgm | Fuzzy search | Search |
| uuid-ossp | UUID generation | Data types |
| pgcrypto | Encryption | Security |
| hstore | Key-value store | Data types |
| ltree | Hierarchical data | Data types |
| PostGIS | Geospatial | Specialized |
| TimescaleDB | Time-series | Specialized |
| pgvector | Vector similarity | AI/ML |
| pg_cron | Job scheduling | Administration |
| pg_partman | Partition management | Administration |
| postgres_fdw | PostgreSQL connection | FDW |
| file_fdw | File access | FDW |

---

## Summary

```sql
-- Extension lifecycle
CREATE EXTENSION name;           -- Install
ALTER EXTENSION name UPDATE;     -- Upgrade
DROP EXTENSION name;             -- Remove

-- FDW setup
CREATE EXTENSION fdw_name;                    -- Install FDW
CREATE SERVER server_name FOREIGN DATA WRAPPER fdw_name OPTIONS (...);
CREATE USER MAPPING FOR user SERVER server OPTIONS (...);
CREATE FOREIGN TABLE table_name (...) SERVER server OPTIONS (...);
```

---

## Further Reading

- PostgreSQL Extension Network (PGXN)
- PostgreSQL Contrib Modules
- Writing PostgreSQL Extensions tutorial
