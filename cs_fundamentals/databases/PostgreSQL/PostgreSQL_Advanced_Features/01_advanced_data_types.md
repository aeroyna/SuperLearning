# Advanced Data Types

## Learning Objectives
- Master PostgreSQL's array type operations
- Understand and use range types effectively
- Create and work with composite types
- Implement enums and domains for data integrity

---

## 1. Array Types

### Creating Arrays

```sql
-- Array column definition
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[],                    -- Array of text
    prices NUMERIC(10,2)[],         -- Array of numeric
    matrix INTEGER[][]              -- Multi-dimensional array
);

-- Insert array data
INSERT INTO products (name, tags, prices) VALUES
    ('Laptop', ARRAY['electronics', 'computers', 'portable'], ARRAY[999.99, 899.99]),
    ('Phone', '{mobile, electronics, gadgets}', '{699.99, 599.99}'),  -- Alternative syntax
    ('Tablet', ARRAY['electronics', 'portable'], ARRAY[499.99]);

-- Array literal syntax
SELECT ARRAY[1, 2, 3];              -- {1,2,3}
SELECT ARRAY[[1,2], [3,4]];         -- {{1,2},{3,4}}
SELECT '{1,2,3}'::INTEGER[];        -- Cast string to array
```

### Array Access and Slicing

```sql
-- Access elements (1-indexed!)
SELECT tags[1] FROM products WHERE name = 'Laptop';  -- 'electronics'

-- Array slicing
SELECT tags[1:2] FROM products WHERE name = 'Laptop';  -- {electronics,computers}

-- Array length
SELECT array_length(tags, 1) FROM products;  -- Length of first dimension
SELECT cardinality(tags) FROM products;       -- Total elements

-- Array dimensions
SELECT array_dims(matrix) FROM products;      -- [1:2][1:2]
SELECT array_ndims(matrix) FROM products;     -- 2
```

### Array Operators

```sql
-- Containment
SELECT * FROM products WHERE tags @> ARRAY['electronics'];     -- Contains
SELECT * FROM products WHERE tags <@ ARRAY['electronics', 'computers', 'portable', 'extra'];  -- Contained by

-- Overlap
SELECT * FROM products WHERE tags && ARRAY['mobile', 'portable'];  -- Any element in common

-- Concatenation
SELECT ARRAY[1,2] || ARRAY[3,4];          -- {1,2,3,4}
SELECT ARRAY[1,2] || 3;                    -- {1,2,3}
SELECT 0 || ARRAY[1,2];                    -- {0,1,2}

-- Equality
SELECT ARRAY[1,2,3] = ARRAY[1,2,3];       -- true
```

### Array Functions

```sql
-- Unnest: Convert array to rows
SELECT unnest(ARRAY['a', 'b', 'c']);
-- Result:
-- a
-- b
-- c

-- Array aggregate: Convert rows to array
SELECT array_agg(name) FROM products;
-- {Laptop,Phone,Tablet}

-- Array position
SELECT array_position(ARRAY['a','b','c'], 'b');  -- 2

-- Array remove
SELECT array_remove(ARRAY[1,2,3,2,1], 2);  -- {1,3,1}

-- Array replace
SELECT array_replace(ARRAY[1,2,3], 2, 10);  -- {1,10,3}

-- Array append/prepend
SELECT array_append(ARRAY[1,2], 3);         -- {1,2,3}
SELECT array_prepend(0, ARRAY[1,2]);        -- {0,1,2}

-- Array to string
SELECT array_to_string(ARRAY['a','b','c'], ', ');  -- 'a, b, c'

-- String to array
SELECT string_to_array('a,b,c', ',');       -- {a,b,c}
```

### Indexing Arrays

```sql
-- GIN index for array containment queries
CREATE INDEX idx_products_tags ON products USING GIN (tags);

-- Query using index
SELECT * FROM products WHERE tags @> ARRAY['electronics'];
SELECT * FROM products WHERE tags && ARRAY['mobile', 'computers'];

-- GIN operator classes
-- array_ops (default): @>, <@, &&, =
```

---

## 2. Range Types

### Built-in Range Types

```sql
-- Integer range
SELECT int4range(1, 10);          -- [1,10)
SELECT int4range(1, 10, '[]');    -- [1,10]  (inclusive both ends)
SELECT int4range(1, 10, '()');    -- (1,10)  (exclusive both ends)

-- Numeric range
SELECT numrange(1.5, 5.5);        -- [1.5,5.5)

-- Date range
SELECT daterange('2024-01-01', '2024-12-31');  -- [2024-01-01,2024-12-31)

-- Timestamp range
SELECT tsrange('2024-01-01 00:00', '2024-01-01 23:59');

-- Timestamp with timezone range
SELECT tstzrange('2024-01-01 00:00+00', '2024-01-01 23:59+00');
```

### Range in Tables

```sql
-- Room booking system
CREATE TABLE room_bookings (
    id SERIAL PRIMARY KEY,
    room_id INTEGER,
    booking_period TSTZRANGE,
    guest_name VARCHAR(100),

    -- Prevent overlapping bookings for same room
    EXCLUDE USING GIST (room_id WITH =, booking_period WITH &&)
);

-- Insert bookings
INSERT INTO room_bookings (room_id, booking_period, guest_name) VALUES
    (1, '[2024-01-15 14:00, 2024-01-18 11:00)', 'John'),
    (1, '[2024-01-20 14:00, 2024-01-22 11:00)', 'Jane');

-- This would fail (overlapping):
-- INSERT INTO room_bookings (room_id, booking_period, guest_name) VALUES
--     (1, '[2024-01-17 14:00, 2024-01-19 11:00)', 'Bob');
```

### Range Operators

```sql
-- Containment
SELECT int4range(1, 10) @> 5;              -- true (contains element)
SELECT int4range(1, 10) @> int4range(3, 7); -- true (contains range)

-- Contained by
SELECT 5 <@ int4range(1, 10);              -- true
SELECT int4range(3, 7) <@ int4range(1, 10); -- true

-- Overlap
SELECT int4range(1, 5) && int4range(3, 8); -- true

-- Adjacent
SELECT int4range(1, 5) -|- int4range(5, 10); -- true

-- Strictly left/right
SELECT int4range(1, 5) << int4range(10, 20); -- true (strictly left)
SELECT int4range(10, 20) >> int4range(1, 5); -- true (strictly right)

-- Union, intersection, difference
SELECT int4range(1, 5) + int4range(3, 8);  -- [1,8)
SELECT int4range(1, 8) * int4range(3, 10); -- [3,8)
SELECT int4range(1, 10) - int4range(3, 7); -- Error (not contiguous)
```

### Range Functions

```sql
-- Bounds
SELECT lower(int4range(1, 10));            -- 1
SELECT upper(int4range(1, 10));            -- 10
SELECT lower_inc(int4range(1, 10));        -- true
SELECT upper_inc(int4range(1, 10));        -- false

-- Empty and infinite
SELECT isempty(int4range(1, 1));           -- true
SELECT lower_inf(int4range(NULL, 10));     -- true (unbounded lower)
SELECT upper_inf(int4range(1, NULL));      -- true (unbounded upper)

-- Range merge
SELECT range_merge(int4range(1, 5), int4range(8, 12)); -- [1,12)
```

### Custom Range Types

```sql
-- Create custom range type
CREATE TYPE float8range AS RANGE (
    SUBTYPE = float8,
    SUBTYPE_DIFF = float8mi  -- For GiST index support
);

-- Use custom range
SELECT '[1.5, 3.5]'::float8range;
```

---

## 3. Composite Types

### Creating Composite Types

```sql
-- Define composite type
CREATE TYPE address AS (
    street VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(2),
    zip VARCHAR(10)
);

CREATE TYPE contact_info AS (
    phone VARCHAR(20),
    email VARCHAR(100),
    address address  -- Nested composite
);

-- Use in table
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    billing_address address,
    shipping_address address,
    contact contact_info
);
```

### Working with Composites

```sql
-- Insert composite data
INSERT INTO customers (name, billing_address, shipping_address) VALUES
    ('John Doe',
     ROW('123 Main St', 'NYC', 'NY', '10001'),
     ROW('456 Oak Ave', 'LA', 'CA', '90001'));

-- Alternative syntax
INSERT INTO customers (name, billing_address) VALUES
    ('Jane Doe', ('789 Pine Rd', 'Chicago', 'IL', '60601'));

-- Access composite fields
SELECT (billing_address).city FROM customers;
SELECT name, (billing_address).* FROM customers;

-- Update composite field
UPDATE customers
SET billing_address.zip = '10002'
WHERE name = 'John Doe';

-- Compare composites
SELECT * FROM customers
WHERE billing_address = ROW('123 Main St', 'NYC', 'NY', '10001')::address;
```

### Row Types

```sql
-- Every table has an implicit row type
CREATE TABLE inventory (
    id SERIAL PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INTEGER
);

-- Use table row type
CREATE TABLE inventory_history (
    id SERIAL PRIMARY KEY,
    inventory_row inventory,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Insert using row type
INSERT INTO inventory_history (inventory_row)
SELECT i FROM inventory i WHERE id = 1;
```

---

## 4. Enumerated Types

### Creating Enums

```sql
-- Define enum type
CREATE TYPE order_status AS ENUM (
    'pending',
    'processing',
    'shipped',
    'delivered',
    'cancelled'
);

CREATE TYPE priority_level AS ENUM ('low', 'medium', 'high', 'urgent');

-- Use in table
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    status order_status DEFAULT 'pending',
    priority priority_level DEFAULT 'medium',
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Enum Operations

```sql
-- Insert with enum
INSERT INTO orders (customer_id, status, priority) VALUES
    (1, 'pending', 'high'),
    (2, 'shipped', 'low');

-- Query with enum
SELECT * FROM orders WHERE status = 'pending';
SELECT * FROM orders WHERE status > 'processing';  -- Ordered comparison

-- Enum ordering (based on definition order)
SELECT * FROM orders ORDER BY status;  -- pending < processing < shipped...

-- Get all enum values
SELECT enum_range(NULL::order_status);
-- {pending,processing,shipped,delivered,cancelled}

-- Get enum position
SELECT enum_range(NULL::order_status);
```

### Modifying Enums

```sql
-- Add new value (at end)
ALTER TYPE order_status ADD VALUE 'refunded';

-- Add value at specific position
ALTER TYPE order_status ADD VALUE 'confirmed' AFTER 'pending';
ALTER TYPE order_status ADD VALUE 'picked' BEFORE 'shipped';

-- Rename value (PostgreSQL 10+)
ALTER TYPE order_status RENAME VALUE 'cancelled' TO 'canceled';

-- Note: Cannot remove enum values directly
-- Must recreate the type (with migration)
```

---

## 5. Domain Types

### Creating Domains

```sql
-- Domain with constraint
CREATE DOMAIN email AS VARCHAR(255)
    CHECK (VALUE ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z]{2,}$');

CREATE DOMAIN positive_integer AS INTEGER
    CHECK (VALUE > 0);

CREATE DOMAIN percentage AS NUMERIC(5,2)
    CHECK (VALUE >= 0 AND VALUE <= 100);

CREATE DOMAIN us_phone AS VARCHAR(20)
    CHECK (VALUE ~ '^\d{3}-\d{3}-\d{4}$');

-- Use in table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email email NOT NULL,
    age positive_integer,
    discount percentage DEFAULT 0
);
```

### Domain Operations

```sql
-- Insert with domain validation
INSERT INTO users (email, age, discount) VALUES
    ('john@example.com', 25, 10.5);  -- OK

-- This would fail:
-- INSERT INTO users (email, age, discount) VALUES
--     ('invalid-email', -5, 150);

-- Modify domain
ALTER DOMAIN positive_integer SET DEFAULT 1;
ALTER DOMAIN positive_integer DROP CONSTRAINT positive_integer_check;
ALTER DOMAIN positive_integer ADD CONSTRAINT positive_check CHECK (VALUE > 0);

-- Rename domain
ALTER DOMAIN us_phone RENAME TO phone_number;
```

---

## 6. Network Types

### IP Address Types

```sql
-- inet: IP address with optional subnet
-- cidr: IP network (subnet)
-- macaddr: MAC address

CREATE TABLE network_devices (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    ip_address INET,
    subnet CIDR,
    mac_address MACADDR
);

INSERT INTO network_devices (name, ip_address, subnet, mac_address) VALUES
    ('Server1', '192.168.1.100/24', '192.168.1.0/24', '08:00:2b:01:02:03'),
    ('Router', '10.0.0.1', '10.0.0.0/8', '00:1a:2b:3c:4d:5e');
```

### Network Operators

```sql
-- Containment
SELECT * FROM network_devices WHERE ip_address << '192.168.0.0/16'::cidr;  -- Contained in

-- Host/network extraction
SELECT host(ip_address), network(ip_address) FROM network_devices;

-- Netmask
SELECT netmask(ip_address) FROM network_devices;

-- Broadcast
SELECT broadcast(ip_address) FROM network_devices;

-- IP arithmetic
SELECT '192.168.1.100'::inet + 10;  -- 192.168.1.110
```

---

## 7. Geometric Types

### Basic Types

```sql
-- Point
SELECT point(1, 2);
SELECT '(1,2)'::point;

-- Line segment
SELECT lseg(point(0,0), point(10,10));

-- Box (rectangle)
SELECT box(point(0,0), point(10,10));

-- Path (open or closed)
SELECT path('[(0,0),(1,1),(2,0)]');    -- Open path
SELECT path('((0,0),(1,1),(2,0))');    -- Closed path

-- Polygon
SELECT polygon('((0,0),(4,0),(4,3),(0,3))');

-- Circle
SELECT circle(point(0,0), 5);           -- Center and radius
```

### Geometric Operations

```sql
-- Distance
SELECT point(0,0) <-> point(3,4);       -- 5

-- Containment
SELECT box('(0,0),(10,10)') @> point(5,5);  -- true

-- Overlap
SELECT box('(0,0),(10,10)') && box('(5,5),(15,15)');  -- true

-- Area and length
SELECT area(circle(point(0,0), 5));     -- ~78.54
SELECT length(lseg('[(0,0),(3,4)]'));   -- 5
```

---

## 8. Practical Examples

### Tags System with Arrays

```sql
-- Blog posts with tags
CREATE TABLE blog_posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    tags TEXT[],
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_posts_tags ON blog_posts USING GIN (tags);

-- Find posts with specific tag
SELECT * FROM blog_posts WHERE 'postgresql' = ANY(tags);

-- Find posts with all specified tags
SELECT * FROM blog_posts WHERE tags @> ARRAY['postgresql', 'tutorial'];

-- Find posts with any of specified tags
SELECT * FROM blog_posts WHERE tags && ARRAY['postgresql', 'mysql'];

-- Tag cloud query
SELECT tag, COUNT(*) as count
FROM blog_posts, unnest(tags) AS tag
GROUP BY tag
ORDER BY count DESC;
```

### Event Scheduling with Ranges

```sql
-- Conference schedule
CREATE TABLE conference_sessions (
    id SERIAL PRIMARY KEY,
    room_id INTEGER,
    title VARCHAR(200),
    time_slot TSTZRANGE,

    EXCLUDE USING GIST (room_id WITH =, time_slot WITH &&)
);

-- Find available rooms for a time period
SELECT DISTINCT room_id
FROM generate_series(1, 10) AS room_id
WHERE room_id NOT IN (
    SELECT room_id FROM conference_sessions
    WHERE time_slot && '[2024-03-15 09:00, 2024-03-15 10:00)'::tstzrange
);

-- Find schedule gaps
SELECT room_id,
       upper(time_slot) AS gap_start,
       lower(LEAD(time_slot) OVER (PARTITION BY room_id ORDER BY time_slot)) AS gap_end
FROM conference_sessions;
```

---

## Summary

| Type | Use Case | Index |
|------|----------|-------|
| Array | Lists, tags, multi-value | GIN |
| Range | Schedules, versions, prices | GiST |
| Composite | Structured data grouping | B-tree (limited) |
| Enum | Fixed set of values | B-tree |
| Domain | Validated subtypes | Inherits from base |
| Network | IP/MAC addresses | GiST |
| Geometric | Spatial data (basic) | GiST |

---

## Further Reading

- PostgreSQL Data Type documentation
- PostgreSQL Array Functions
- Range Types documentation
