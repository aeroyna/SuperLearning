# Data Migration Patterns

## Schema vs Data Migrations

```
┌─────────────────────────────────────────────────────────────────┐
│              Types of Migrations                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SCHEMA MIGRATIONS                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • CREATE/ALTER/DROP tables                                 │ │
│  │ • Add/remove columns                                       │ │
│  │ • Add/remove indexes                                       │ │
│  │ • Modify constraints                                       │ │
│  │                                                             │ │
│  │ Usually fast, can be transactional                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATA MIGRATIONS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Backfill new columns                                     │ │
│  │ • Transform existing data                                  │ │
│  │ • Move data between tables                                 │ │
│  │ • Denormalization/normalization                            │ │
│  │                                                             │ │
│  │ Can be slow, may need batching                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMBINED (Use Carefully)                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Add column then backfill                                │ │
│  │ ALTER TABLE users ADD COLUMN full_name VARCHAR(200);       │ │
│  │ UPDATE users SET full_name = first_name || ' ' || last_name;│ │
│  │                                                             │ │
│  │ Risk: Large UPDATE can lock table                         │ │
│  │ Better: Separate into two migrations                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Backfill Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│              Backfilling Data                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SMALL TABLES (< 100K rows)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Simple UPDATE is fine                                   │ │
│  │ UPDATE users SET status = 'active' WHERE status IS NULL;   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MEDIUM TABLES (100K - 10M rows)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Batch updates                                           │ │
│  │ DO $$                                                       │ │
│  │ DECLARE                                                     │ │
│  │   batch_size INT := 10000;                                 │ │
│  │   updated INT;                                              │ │
│  │ BEGIN                                                       │ │
│  │   LOOP                                                      │ │
│  │     UPDATE users                                            │ │
│  │     SET status = 'active'                                  │ │
│  │     WHERE id IN (                                          │ │
│  │       SELECT id FROM users                                 │ │
│  │       WHERE status IS NULL                                 │ │
│  │       LIMIT batch_size                                     │ │
│  │       FOR UPDATE SKIP LOCKED                               │ │
│  │     );                                                      │ │
│  │                                                             │ │
│  │     GET DIAGNOSTICS updated = ROW_COUNT;                   │ │
│  │     EXIT WHEN updated = 0;                                 │ │
│  │                                                             │ │
│  │     COMMIT;                                                 │ │
│  │     PERFORM pg_sleep(0.1);  -- Pause between batches      │ │
│  │   END LOOP;                                                 │ │
│  │ END $$;                                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  LARGE TABLES (> 10M rows)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Options:                                                    │ │
│  │ 1. Background job (Ruby Sidekiq, Python Celery)           │ │
│  │ 2. Streaming with pg_dump/COPY                            │ │
│  │ 3. Create new table, swap                                  │ │
│  │                                                             │ │
│  │ -- New table approach                                      │ │
│  │ CREATE TABLE users_new AS                                  │ │
│  │ SELECT *, COALESCE(status, 'active') as status_new        │ │
│  │ FROM users;                                                 │ │
│  │                                                             │ │
│  │ -- Then swap tables during maintenance window              │ │
│  │ ALTER TABLE users RENAME TO users_old;                     │ │
│  │ ALTER TABLE users_new RENAME TO users;                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Data Transformation Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│              Common Data Transformations                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SPLIT COLUMN                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- From: full_name                                         │ │
│  │ -- To: first_name, last_name                               │ │
│  │                                                             │ │
│  │ ALTER TABLE users ADD COLUMN first_name VARCHAR(100);      │ │
│  │ ALTER TABLE users ADD COLUMN last_name VARCHAR(100);       │ │
│  │                                                             │ │
│  │ UPDATE users SET                                            │ │
│  │   first_name = split_part(full_name, ' ', 1),              │ │
│  │   last_name = split_part(full_name, ' ', 2);               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MERGE COLUMNS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Combine first_name and last_name                        │ │
│  │ ALTER TABLE users ADD COLUMN display_name VARCHAR(200);    │ │
│  │                                                             │ │
│  │ UPDATE users SET                                            │ │
│  │   display_name = CONCAT(first_name, ' ', last_name);       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NORMALIZE (Extract to New Table)                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- From: orders.customer_name, orders.customer_email       │ │
│  │ -- To: customers table with foreign key                   │ │
│  │                                                             │ │
│  │ CREATE TABLE customers (                                   │ │
│  │   id SERIAL PRIMARY KEY,                                   │ │
│  │   name VARCHAR(100),                                       │ │
│  │   email VARCHAR(255) UNIQUE                                │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ INSERT INTO customers (name, email)                        │ │
│  │ SELECT DISTINCT customer_name, customer_email              │ │
│  │ FROM orders;                                                │ │
│  │                                                             │ │
│  │ ALTER TABLE orders ADD COLUMN customer_id INT;             │ │
│  │                                                             │ │
│  │ UPDATE orders o SET customer_id = c.id                     │ │
│  │ FROM customers c                                            │ │
│  │ WHERE o.customer_email = c.email;                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DENORMALIZE (Embed Data)                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Cache customer name in orders for faster reads         │ │
│  │ ALTER TABLE orders ADD COLUMN customer_name VARCHAR(100);  │ │
│  │                                                             │ │
│  │ UPDATE orders o SET customer_name = c.name                 │ │
│  │ FROM customers c                                            │ │
│  │ WHERE o.customer_id = c.id;                                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Handling Dependencies

```
┌─────────────────────────────────────────────────────────────────┐
│              Migration Dependencies                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FOREIGN KEY CONSIDERATIONS                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Problem: Can't add FK if data violates constraint      │ │
│  │ -- Solution: Clean data first                              │ │
│  │                                                             │ │
│  │ -- Step 1: Find orphaned records                           │ │
│  │ SELECT o.id FROM orders o                                  │ │
│  │ LEFT JOIN customers c ON o.customer_id = c.id              │ │
│  │ WHERE c.id IS NULL;                                        │ │
│  │                                                             │ │
│  │ -- Step 2: Fix or delete orphans                          │ │
│  │ DELETE FROM orders WHERE customer_id NOT IN                │ │
│  │   (SELECT id FROM customers);                              │ │
│  │                                                             │ │
│  │ -- Step 3: Add constraint                                  │ │
│  │ ALTER TABLE orders                                          │ │
│  │ ADD CONSTRAINT fk_customer                                  │ │
│  │ FOREIGN KEY (customer_id) REFERENCES customers(id);        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DEPENDENT MIGRATIONS                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Migration A: Create lookup table                           │ │
│  │ Migration B: Add FK to lookup table (depends on A)        │ │
│  │ Migration C: Backfill FK values (depends on B)            │ │
│  │                                                             │ │
│  │ Run in order: A → B → C                                   │ │
│  │ Can't parallelize dependent migrations                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATA VALIDATION                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Verify migration succeeded                              │ │
│  │ -- Add to migration or run separately                     │ │
│  │                                                             │ │
│  │ DO $$                                                       │ │
│  │ DECLARE                                                     │ │
│  │   null_count INT;                                          │ │
│  │ BEGIN                                                       │ │
│  │   SELECT COUNT(*) INTO null_count                          │ │
│  │   FROM users WHERE status IS NULL;                         │ │
│  │                                                             │ │
│  │   IF null_count > 0 THEN                                   │ │
│  │     RAISE EXCEPTION 'Migration incomplete: % null values', │ │
│  │       null_count;                                          │ │
│  │   END IF;                                                   │ │
│  │ END $$;                                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
