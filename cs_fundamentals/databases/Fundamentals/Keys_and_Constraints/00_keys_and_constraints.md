# Keys and Constraints

## Overview

Keys and constraints are fundamental to maintaining data integrity in relational databases. They enforce rules about what data can be stored and how tables relate to each other.

## Topics Covered

1. **[Primary Keys](01_primary_keys.md)** - Unique identification of rows
2. **[Foreign Keys and Referential Integrity](02_foreign_keys_and_referential_integrity.md)** - Relationships between tables
3. **[Unique and Check Constraints](03_unique_and_check_constraints.md)** - Data validation rules
4. **[Composite Keys and Surrogate Keys](04_composite_and_surrogate_keys.md)** - Key design strategies

## Keys and Constraints Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      KEYS AND CONSTRAINTS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   KEYS (Identification)                   CONSTRAINTS (Validation)           │
│   ─────────────────────                   ────────────────────────           │
│   • Primary Key         ──────────────▶   • NOT NULL                        │
│   • Foreign Key                           • UNIQUE                          │
│   • Candidate Key                         • CHECK                           │
│   • Alternate Key                         • DEFAULT                         │
│   • Composite Key                         • EXCLUSION (PostgreSQL)          │
│   • Surrogate Key                                                           │
│                                                                              │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │                        EXAMPLE SCHEMA                                 │ │
│   ├───────────────────────────────────────────────────────────────────────┤ │
│   │  CREATE TABLE orders (                                                │ │
│   │      id SERIAL PRIMARY KEY,           -- Primary Key                  │ │
│   │      customer_id INT NOT NULL,        -- NOT NULL + Foreign Key       │ │
│   │      order_number VARCHAR(50) UNIQUE, -- Unique constraint            │ │
│   │      total DECIMAL CHECK (total >= 0),-- Check constraint             │ │
│   │      status VARCHAR DEFAULT 'pending',-- Default constraint           │ │
│   │      FOREIGN KEY (customer_id) REFERENCES customers(id)               │ │
│   │  );                                                                   │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Quick Reference

```sql
-- Primary Key
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL
);

-- Foreign Key with actions
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- Check constraint
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price DECIMAL(10,2) CHECK (price > 0),
    quantity INT CHECK (quantity >= 0)
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id)
);
```

## Learning Objectives

After completing this section, you will be able to:
- Design effective primary key strategies
- Implement referential integrity with foreign keys
- Use constraints to enforce business rules
- Choose between natural and surrogate keys
- Handle constraint violations gracefully
