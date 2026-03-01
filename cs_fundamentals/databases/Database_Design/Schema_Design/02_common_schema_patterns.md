# Common Schema Patterns

## Entity-Attribute-Value (EAV)

```
┌─────────────────────────────────────────────────────────────────┐
│              Entity-Attribute-Value Pattern                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRADITIONAL APPROACH (Fixed Schema)                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ products                                                    │ │
│  │ ─────────────────────────────────────────────────────────  │ │
│  │ id │ name   │ color │ size │ weight │ material │ ...      │ │
│  │                                                             │ │
│  │ Problem: Different products have different attributes      │ │
│  │ - Electronics: voltage, wattage                            │ │
│  │ - Clothing: size, color, fabric                            │ │
│  │ - Books: author, ISBN, pages                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  EAV APPROACH                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ products               product_attributes                  │ │
│  │ ────────────────       ─────────────────────────────────── │ │
│  │ id │ name │ type       product_id │ attribute │ value     │ │
│  │ 1  │ iPhone│ phone     1          │ color     │ silver    │ │
│  │ 2  │ T-Shirt│ clothing 1          │ storage   │ 128GB     │ │
│  │                        2          │ size      │ M         │ │
│  │                        2          │ color     │ blue      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PROS AND CONS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Flexible schema                                          │ │
│  │ ✓ No schema migrations for new attributes                 │ │
│  │ ✓ Sparse data handled efficiently                         │ │
│  │                                                             │ │
│  │ ✗ Complex queries                                          │ │
│  │ ✗ No type safety                                           │ │
│  │ ✗ No referential integrity on values                      │ │
│  │ ✗ Performance issues with many attributes                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MODERN ALTERNATIVE: JSONB                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE products (                                    │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     name VARCHAR(255),                                     │ │
│  │     type VARCHAR(50),                                      │ │
│  │     attributes JSONB DEFAULT '{}'                          │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Query with JSON                                         │ │
│  │ SELECT * FROM products                                     │ │
│  │ WHERE attributes->>'color' = 'silver';                     │ │
│  │                                                             │ │
│  │ -- Index for JSON queries                                  │ │
│  │ CREATE INDEX idx_products_attrs ON products                │ │
│  │ USING GIN(attributes);                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Self-Referential Relationships

```
┌─────────────────────────────────────────────────────────────────┐
│              Hierarchical Data Patterns                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ADJACENCY LIST (Parent Reference)                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE categories (                                  │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     name VARCHAR(100),                                     │ │
│  │     parent_id BIGINT REFERENCES categories(id)             │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ id │ name          │ parent_id                             │ │
│  │ ───┼───────────────┼───────────                            │ │
│  │ 1  │ Electronics   │ NULL                                  │ │
│  │ 2  │ Computers     │ 1                                     │ │
│  │ 3  │ Laptops       │ 2                                     │ │
│  │ 4  │ Gaming Laptop │ 3                                     │ │
│  │                                                             │ │
│  │ ✓ Simple to implement                                      │ │
│  │ ✓ Easy to move subtrees                                    │ │
│  │ ✗ Recursive queries needed for ancestors/descendants      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MATERIALIZED PATH                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE categories (                                  │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     name VARCHAR(100),                                     │ │
│  │     path VARCHAR(255)  -- e.g., '1/2/3/4'                 │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ id │ name          │ path                                  │ │
│  │ ───┼───────────────┼─────────                              │ │
│  │ 1  │ Electronics   │ 1                                     │ │
│  │ 2  │ Computers     │ 1/2                                   │ │
│  │ 3  │ Laptops       │ 1/2/3                                 │ │
│  │ 4  │ Gaming Laptop │ 1/2/3/4                               │ │
│  │                                                             │ │
│  │ -- Find all descendants                                    │ │
│  │ SELECT * FROM categories WHERE path LIKE '1/2/%';          │ │
│  │                                                             │ │
│  │ ✓ Fast ancestor/descendant queries                        │ │
│  │ ✗ Path updates needed when moving nodes                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NESTED SETS                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE categories (                                  │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     name VARCHAR(100),                                     │ │
│  │     lft INT,   -- left boundary                           │ │
│  │     rgt INT    -- right boundary                          │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Tree visualization:                                     │ │
│  │ --       Electronics (1,8)                                 │ │
│  │ --          │                                               │ │
│  │ --       Computers (2,7)                                   │ │
│  │ --          │                                               │ │
│  │ --       Laptops (3,6)                                     │ │
│  │ --          │                                               │ │
│  │ --    Gaming Laptop (4,5)                                  │ │
│  │                                                             │ │
│  │ ✓ Fast subtree queries                                    │ │
│  │ ✗ Expensive inserts/moves (renumber many rows)            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CLOSURE TABLE                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE category_closure (                            │ │
│  │     ancestor_id BIGINT,                                    │ │
│  │     descendant_id BIGINT,                                  │ │
│  │     depth INT,                                             │ │
│  │     PRIMARY KEY (ancestor_id, descendant_id)               │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- All paths stored explicitly                             │ │
│  │ ancestor │ descendant │ depth                              │ │
│  │ ─────────┼────────────┼──────                              │ │
│  │ 1        │ 1          │ 0                                  │ │
│  │ 1        │ 2          │ 1                                  │ │
│  │ 1        │ 3          │ 2                                  │ │
│  │ 2        │ 2          │ 0                                  │ │
│  │ 2        │ 3          │ 1                                  │ │
│  │                                                             │ │
│  │ ✓ Fast all queries (ancestors, descendants)               │ │
│  │ ✓ Easy subtree operations                                 │ │
│  │ ✗ More storage space                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Polymorphic Associations

```
┌─────────────────────────────────────────────────────────────────┐
│              Polymorphic Relationship Patterns                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PROBLEM: Comments can belong to posts, photos, or videos      │
│                                                                  │
│  OPTION 1: SEPARATE TABLES                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ post_comments                                               │ │
│  │   id, post_id, content, user_id                            │ │
│  │                                                             │ │
│  │ photo_comments                                              │ │
│  │   id, photo_id, content, user_id                           │ │
│  │                                                             │ │
│  │ ✓ Referential integrity                                    │ │
│  │ ✗ Code duplication                                         │ │
│  │ ✗ Hard to query all comments                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  OPTION 2: TYPE COLUMN (Polymorphic)                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE comments (                                    │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     commentable_type VARCHAR(50),  -- 'post', 'photo'     │ │
│  │     commentable_id BIGINT,                                 │ │
│  │     content TEXT,                                          │ │
│  │     user_id BIGINT                                         │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ CREATE INDEX idx_comments_poly                             │ │
│  │ ON comments(commentable_type, commentable_id);             │ │
│  │                                                             │ │
│  │ ✓ Single table for all comments                           │ │
│  │ ✓ Easy to query all comments                              │ │
│  │ ✗ No foreign key constraint                               │ │
│  │ ✗ Type safety relies on application                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  OPTION 3: MULTIPLE FOREIGN KEYS                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE comments (                                    │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     post_id BIGINT REFERENCES posts(id),                   │ │
│  │     photo_id BIGINT REFERENCES photos(id),                 │ │
│  │     video_id BIGINT REFERENCES videos(id),                 │ │
│  │     content TEXT,                                          │ │
│  │     CONSTRAINT one_parent CHECK (                          │ │
│  │         (post_id IS NOT NULL)::INT +                       │ │
│  │         (photo_id IS NOT NULL)::INT +                      │ │
│  │         (video_id IS NOT NULL)::INT = 1                    │ │
│  │     )                                                       │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ ✓ Referential integrity                                    │ │
│  │ ✗ Many nullable columns                                   │ │
│  │ ✗ Schema changes for new types                            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## State Machines

```
┌─────────────────────────────────────────────────────────────────┐
│              State Machine Pattern                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ORDER STATE MACHINE                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │    pending ──► confirmed ──► shipped ──► delivered         │ │
│  │       │            │             │                          │ │
│  │       └────────────┴─────────────┴────► cancelled          │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SCHEMA DESIGN                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TYPE order_status AS ENUM (                         │ │
│  │     'pending', 'confirmed', 'shipped',                     │ │
│  │     'delivered', 'cancelled'                               │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ CREATE TABLE orders (                                      │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     status order_status DEFAULT 'pending',                 │ │
│  │     -- timestamps for each state                           │ │
│  │     confirmed_at TIMESTAMP,                                │ │
│  │     shipped_at TIMESTAMP,                                  │ │
│  │     delivered_at TIMESTAMP,                                │ │
│  │     cancelled_at TIMESTAMP                                 │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- State transition history                                │ │
│  │ CREATE TABLE order_status_history (                        │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     order_id BIGINT REFERENCES orders(id),                 │ │
│  │     from_status order_status,                              │ │
│  │     to_status order_status,                                │ │
│  │     changed_by BIGINT,                                     │ │
│  │     changed_at TIMESTAMP DEFAULT NOW(),                    │ │
│  │     reason TEXT                                            │ │
│  │ );                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  VALID TRANSITIONS (Application or DB constraint)              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE valid_transitions (                           │ │
│  │     from_status order_status,                              │ │
│  │     to_status order_status,                                │ │
│  │     PRIMARY KEY (from_status, to_status)                   │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ INSERT INTO valid_transitions VALUES                       │ │
│  │     ('pending', 'confirmed'),                              │ │
│  │     ('pending', 'cancelled'),                              │ │
│  │     ('confirmed', 'shipped'),                              │ │
│  │     ('confirmed', 'cancelled'),                            │ │
│  │     ('shipped', 'delivered'),                              │ │
│  │     ('shipped', 'cancelled');                              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Soft Deletes

```
┌─────────────────────────────────────────────────────────────────┐
│              Soft Delete Pattern                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  IMPLEMENTATION                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE users (                                       │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     email VARCHAR(255) UNIQUE,                             │ │
│  │     name VARCHAR(100),                                     │ │
│  │     deleted_at TIMESTAMP NULL  -- NULL = not deleted       │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Partial index for active records                        │ │
│  │ CREATE UNIQUE INDEX idx_users_email_active                 │ │
│  │ ON users(email) WHERE deleted_at IS NULL;                  │ │
│  │                                                             │ │
│  │ -- View for easy querying                                  │ │
│  │ CREATE VIEW active_users AS                                │ │
│  │ SELECT * FROM users WHERE deleted_at IS NULL;              │ │
│  │                                                             │ │
│  │ -- "Delete" operation                                      │ │
│  │ UPDATE users SET deleted_at = NOW() WHERE id = ?;          │ │
│  │                                                             │ │
│  │ -- Restore operation                                       │ │
│  │ UPDATE users SET deleted_at = NULL WHERE id = ?;           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CONSIDERATIONS                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Data recovery possible                                   │ │
│  │ ✓ Audit trail preserved                                    │ │
│  │ ✓ Referential integrity maintained                         │ │
│  │                                                             │ │
│  │ ✗ Must filter deleted in all queries                      │ │
│  │ ✗ Unique constraints need special handling                │ │
│  │ ✗ Table grows without hard deletes                        │ │
│  │                                                             │ │
│  │ Best practice: Periodically archive old soft-deleted rows │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
