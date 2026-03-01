# Schema Migration Strategies

## State-Based vs Migration-Based

```
┌─────────────────────────────────────────────────────────────────┐
│              Two Approaches to Schema Management                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MIGRATION-BASED (Incremental)                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- V1: Create initial schema                               │ │
│  │ CREATE TABLE users (id INT, name VARCHAR(100));            │ │
│  │                                                             │ │
│  │ -- V2: Add email column                                    │ │
│  │ ALTER TABLE users ADD COLUMN email VARCHAR(255);           │ │
│  │                                                             │ │
│  │ -- V3: Add index                                           │ │
│  │ CREATE INDEX idx_users_email ON users(email);              │ │
│  │                                                             │ │
│  │ Apply migrations in sequence: V1 → V2 → V3                 │ │
│  │                                                             │ │
│  │ ✓ Full history of changes                                  │ │
│  │ ✓ Explicit control over each step                         │ │
│  │ ✓ Rollback scripts possible                                │ │
│  │ ✗ Must maintain all migrations                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STATE-BASED (Declarative)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Define desired end state                                │ │
│  │ CREATE TABLE users (                                       │ │
│  │     id INT PRIMARY KEY,                                    │ │
│  │     name VARCHAR(100),                                     │ │
│  │     email VARCHAR(255)                                     │ │
│  │ );                                                          │ │
│  │ CREATE INDEX idx_users_email ON users(email);              │ │
│  │                                                             │ │
│  │ Tool generates diff and applies changes                    │ │
│  │                                                             │ │
│  │ ✓ Easier to understand current schema                     │ │
│  │ ✓ No migration history to maintain                        │ │
│  │ ✗ Less control over how changes applied                   │ │
│  │ ✗ Harder to handle data migrations                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RECOMMENDATION: Use migration-based for most cases            │
└─────────────────────────────────────────────────────────────────┘
```

## Up and Down Migrations

```
┌─────────────────────────────────────────────────────────────────┐
│              Reversible Migrations                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STRUCTURE                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- up.sql (apply)                                          │ │
│  │ ALTER TABLE users ADD COLUMN phone VARCHAR(20);            │ │
│  │                                                             │ │
│  │ -- down.sql (rollback)                                     │ │
│  │ ALTER TABLE users DROP COLUMN phone;                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  EXAMPLE: Rails Migration                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ class AddPhoneToUsers < ActiveRecord::Migration[7.0]       │ │
│  │   def up                                                    │ │
│  │     add_column :users, :phone, :string                     │ │
│  │   end                                                       │ │
│  │                                                             │ │
│  │   def down                                                  │ │
│  │     remove_column :users, :phone                           │ │
│  │   end                                                       │ │
│  │ end                                                         │ │
│  │                                                             │ │
│  │ # Or use reversible change method                          │ │
│  │ class AddPhoneToUsers < ActiveRecord::Migration[7.0]       │ │
│  │   def change                                                │ │
│  │     add_column :users, :phone, :string                     │ │
│  │   end                                                       │ │
│  │ end                                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  IRREVERSIBLE MIGRATIONS                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Some migrations cannot be reversed:                        │ │
│  │                                                             │ │
│  │ • DROP COLUMN (data lost)                                  │ │
│  │ • Change column type (may lose precision)                  │ │
│  │ • DROP TABLE                                               │ │
│  │                                                             │ │
│  │ class RemoveMiddleName < ActiveRecord::Migration[7.0]      │ │
│  │   def up                                                    │ │
│  │     remove_column :users, :middle_name                     │ │
│  │   end                                                       │ │
│  │                                                             │ │
│  │   def down                                                  │ │
│  │     raise ActiveRecord::IrreversibleMigration              │ │
│  │     # Or recreate column without data                      │ │
│  │   end                                                       │ │
│  │ end                                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Migration Ordering

```
┌─────────────────────────────────────────────────────────────────┐
│              Handling Migration Order                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DEPENDENCY MANAGEMENT                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Migrations must run in order when dependencies exist:     │ │
│  │                                                             │ │
│  │ V1: CREATE TABLE users (...);                              │ │
│  │ V2: CREATE TABLE orders (..., user_id REFERENCES users);  │ │
│  │                                                             │ │
│  │ V2 depends on V1 - must run in sequence                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BRANCH CONFLICTS                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ main: V1 → V2                                              │ │
│  │                                                             │ │
│  │ feature-a: V1 → V2 → V3a (add column A)                   │ │
│  │ feature-b: V1 → V2 → V3b (add column B)                   │ │
│  │                                                             │ │
│  │ Both create V3 - conflict!                                 │ │
│  │                                                             │ │
│  │ Solutions:                                                  │ │
│  │ 1. Timestamp-based naming (20240115... vs 20240116...)    │ │
│  │ 2. Rebase and renumber before merge                       │ │
│  │ 3. Use tools that handle concurrent migrations            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MIGRATION TABLE                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Most tools track applied migrations in a table:           │ │
│  │                                                             │ │
│  │ schema_migrations                                          │ │
│  │ ─────────────────────────────────────────                  │ │
│  │ version          │ applied_at                              │ │
│  │ ─────────────────┼────────────────────────                 │ │
│  │ 20240115120000   │ 2024-01-15 12:05:00                    │ │
│  │ 20240115130000   │ 2024-01-15 13:10:00                    │ │
│  │ 20240116090000   │ 2024-01-16 09:05:00                    │ │
│  │                                                             │ │
│  │ Tool checks this table to know which to run               │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Safe Migration Practices

```
┌─────────────────────────────────────────────────────────────────┐
│              Migration Safety Rules                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ALWAYS                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Test migrations on production-sized data                │ │
│  │ ✓ Have a rollback plan                                    │ │
│  │ ✓ Back up before running                                  │ │
│  │ ✓ Run in transaction when possible                        │ │
│  │ ✓ Monitor locks during migration                          │ │
│  │ ✓ Time migrations in staging                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NEVER                                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✗ Run untested migrations in production                   │ │
│  │ ✗ Modify already-applied migrations                       │ │
│  │ ✗ Delete migration files after applying                   │ │
│  │ ✗ Ignore migration failures                               │ │
│  │ ✗ Mix schema and data changes without care               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TRANSACTION HANDLING                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Wrap in transaction (safe)                              │ │
│  │ BEGIN;                                                      │ │
│  │ ALTER TABLE users ADD COLUMN phone VARCHAR(20);            │ │
│  │ UPDATE users SET phone = 'unknown' WHERE phone IS NULL;   │ │
│  │ COMMIT;                                                     │ │
│  │                                                             │ │
│  │ Note: Some operations can't be in transactions:           │ │
│  │ • CREATE INDEX CONCURRENTLY (PostgreSQL)                  │ │
│  │ • ALTER TABLE ... ADD COLUMN (some cases)                 │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
