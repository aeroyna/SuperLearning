# Zero-Downtime Migrations

## The Challenge

```
┌─────────────────────────────────────────────────────────────────┐
│              Why Zero-Downtime is Hard                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRADITIONAL DEPLOYMENT                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Stop application                                        │ │
│  │ 2. Run migrations                                          │ │
│  │ 3. Deploy new code                                         │ │
│  │ 4. Start application                                       │ │
│  │                                                             │ │
│  │ Simple but causes downtime                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ZERO-DOWNTIME CHALLENGES                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Old and new code run simultaneously                     │ │
│  │ • Schema must work with both versions                     │ │
│  │ • Long-running migrations lock tables                     │ │
│  │ • Data consistency during transition                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMPATIBILITY MATRIX                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Time ──────────────────────────────────────────────►      │ │
│  │                                                             │ │
│  │  Schema V1     Schema V2                                   │ │
│  │  ─────────────────────────                                 │ │
│  │  Code V1 │  ✓  │  ?  │   ← Must work!                     │ │
│  │  Code V2 │  ?  │  ✓  │   ← Must work!                     │ │
│  │  ─────────────────────                                     │ │
│  │                                                             │ │
│  │  During transition, both combinations run                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Expand-Contract Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│              Expand and Contract                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  EXAMPLE: Rename column 'name' to 'full_name'                  │
│                                                                  │
│  DANGEROUS (causes downtime):                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ALTER TABLE users RENAME COLUMN name TO full_name;         │ │
│  │ -- Old code immediately breaks!                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SAFE (expand-contract):                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PHASE 1: EXPAND (add new column)                           │ │
│  │ ─────────────────────────────────                          │ │
│  │ Migration: Add new column                                  │ │
│  │   ALTER TABLE users ADD COLUMN full_name VARCHAR(200);     │ │
│  │                                                             │ │
│  │ Code: Write to both columns                                │ │
│  │   user.name = value                                        │ │
│  │   user.full_name = value                                   │ │
│  │                                                             │ │
│  │ Backfill: Copy existing data                               │ │
│  │   UPDATE users SET full_name = name WHERE full_name IS NULL;│ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PHASE 2: MIGRATE (switch to new column)                    │ │
│  │ ─────────────────────────────────                          │ │
│  │ Code: Read from new column                                 │ │
│  │   display(user.full_name)                                  │ │
│  │                                                             │ │
│  │ Code: Still write to both (for safety)                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PHASE 3: CONTRACT (remove old column)                      │ │
│  │ ─────────────────────────────────                          │ │
│  │ Code: Stop writing to old column                          │ │
│  │                                                             │ │
│  │ Migration: Drop old column                                 │ │
│  │   ALTER TABLE users DROP COLUMN name;                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Common Operations

```
┌─────────────────────────────────────────────────────────────────┐
│              Zero-Downtime Patterns                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ADD COLUMN (Usually Safe)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- PostgreSQL 11+: Instant for nullable columns           │ │
│  │ ALTER TABLE users ADD COLUMN phone VARCHAR(20);            │ │
│  │                                                             │ │
│  │ -- With default (PostgreSQL 11+): Also instant            │ │
│  │ ALTER TABLE users ADD COLUMN status VARCHAR(20)            │ │
│  │     DEFAULT 'active';                                      │ │
│  │                                                             │ │
│  │ Old code: Ignores new column                               │ │
│  │ New code: Uses new column                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DROP COLUMN (Two-Phase)                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Phase 1: Stop using column in code                        │ │
│  │   - Remove all reads                                       │ │
│  │   - Remove all writes                                      │ │
│  │   - Deploy code                                            │ │
│  │                                                             │ │
│  │ Phase 2: Drop column                                       │ │
│  │   ALTER TABLE users DROP COLUMN old_column;                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ADD NOT NULL CONSTRAINT (Three-Phase)                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Phase 1: Add column as nullable                            │ │
│  │   ALTER TABLE users ADD COLUMN email VARCHAR(255);         │ │
│  │                                                             │ │
│  │ Phase 2: Backfill and enforce in code                     │ │
│  │   UPDATE users SET email = 'unknown' WHERE email IS NULL;  │ │
│  │   -- Code validates NOT NULL                               │ │
│  │                                                             │ │
│  │ Phase 3: Add constraint                                    │ │
│  │   ALTER TABLE users ALTER COLUMN email SET NOT NULL;       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ADD INDEX (Concurrently)                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- PostgreSQL: CONCURRENTLY avoids locking                │ │
│  │ CREATE INDEX CONCURRENTLY idx_users_email                  │ │
│  │     ON users(email);                                       │ │
│  │                                                             │ │
│  │ -- Takes longer but doesn't block writes                  │ │
│  │ -- Cannot run in transaction                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CHANGE COLUMN TYPE (Complex)                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Example: INT to BIGINT                                     │ │
│  │                                                             │ │
│  │ 1. Add new column: user_id_new BIGINT                     │ │
│  │ 2. Dual-write to both columns                              │ │
│  │ 3. Backfill: UPDATE ... SET user_id_new = user_id         │ │
│  │ 4. Switch reads to new column                              │ │
│  │ 5. Stop writing old column                                 │ │
│  │ 6. Drop old, rename new                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Lock-Free Operations

```
┌─────────────────────────────────────────────────────────────────┐
│              Avoiding Table Locks                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  POSTGRESQL LOCK LEVELS                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Operation                    │ Lock Level                  │ │
│  │ ─────────────────────────────┼─────────────────────────── │ │
│  │ SELECT                       │ AccessShare (no block)     │ │
│  │ INSERT/UPDATE/DELETE         │ RowExclusive               │ │
│  │ CREATE INDEX                 │ ShareLock (blocks writes)  │ │
│  │ CREATE INDEX CONCURRENTLY    │ ShareUpdateExclusive       │ │
│  │ ALTER TABLE (most)           │ AccessExclusive (blocks all)│ │
│  │ ADD COLUMN (nullable)        │ AccessExclusive (brief)    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  LOCK TIMEOUT                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Set timeout to avoid long waits                        │ │
│  │ SET lock_timeout = '5s';                                   │ │
│  │ ALTER TABLE users ADD COLUMN new_col VARCHAR(50);          │ │
│  │                                                             │ │
│  │ -- If lock can't be acquired in 5s, fails                 │ │
│  │ -- Better than blocking production                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BATCHED UPDATES                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Bad: Locks table for long time                         │ │
│  │ UPDATE users SET status = 'active';                        │ │
│  │                                                             │ │
│  │ -- Good: Batch updates                                     │ │
│  │ DO $$                                                       │ │
│  │ DECLARE batch_size INT := 1000;                            │ │
│  │ BEGIN                                                       │ │
│  │   LOOP                                                      │ │
│  │     UPDATE users SET status = 'active'                     │ │
│  │     WHERE id IN (                                          │ │
│  │       SELECT id FROM users                                 │ │
│  │       WHERE status IS NULL LIMIT batch_size                │ │
│  │     );                                                      │ │
│  │     EXIT WHEN NOT FOUND;                                   │ │
│  │     COMMIT;                                                 │ │
│  │     PERFORM pg_sleep(0.1);  -- Let other queries run      │ │
│  │   END LOOP;                                                 │ │
│  │ END $$;                                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Deployment Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│              Coordinating Code and Schema                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BACKWARD COMPATIBLE FIRST                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Deploy 1: Run migration (schema works with old code)      │ │
│  │ Deploy 2: Deploy new code                                  │ │
│  │ Deploy 3: Cleanup migration (remove old column)           │ │
│  │                                                             │ │
│  │ Timeline:                                                   │ │
│  │ ─────────────────────────────────────────────────►        │ │
│  │ │ Migration │ New Code │ Cleanup │                        │ │
│  │   Schema V2   Schema V2   Schema V3                       │ │
│  │   Code V1     Code V2     Code V2                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  FORWARD COMPATIBLE FIRST                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Deploy 1: Deploy code (works with old and new schema)     │ │
│  │ Deploy 2: Run migration                                    │ │
│  │ Deploy 3: Deploy code (cleanup old code paths)            │ │
│  │                                                             │ │
│  │ Timeline:                                                   │ │
│  │ ─────────────────────────────────────────────────►        │ │
│  │ │ New Code │ Migration │ Cleanup │                        │ │
│  │   Schema V1   Schema V2   Schema V2                       │ │
│  │   Code V2     Code V2     Code V3                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DUAL COMPATIBILITY WINDOW                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ During rolling deployments:                                │ │
│  │ • Some servers run old code                                │ │
│  │ • Some servers run new code                                │ │
│  │ • Schema must work with BOTH                               │ │
│  │                                                             │ │
│  │ Rule: Never remove something old code still uses          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
