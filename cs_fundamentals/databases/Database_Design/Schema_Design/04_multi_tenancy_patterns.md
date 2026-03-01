# Multi-Tenancy Patterns

## Multi-Tenancy Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              Multi-Tenancy Concepts                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DEFINITION                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Multi-tenancy: Single application instance serving         │ │
│  │ multiple customers (tenants) with data isolation           │ │
│  │                                                             │ │
│  │ Each tenant sees only their own data                       │ │
│  │ Shared infrastructure reduces costs                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  KEY CONSIDERATIONS                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Data isolation and security                              │ │
│  │ • Performance isolation (noisy neighbor)                   │ │
│  │ • Customization per tenant                                 │ │
│  │ • Scalability                                              │ │
│  │ • Operational complexity                                   │ │
│  │ • Cost efficiency                                          │ │
│  │ • Compliance requirements                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ISOLATION SPECTRUM                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Shared ◄─────────────────────────────────────► Isolated  │ │
│  │                                                             │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐   │ │
│  │  │ Shared  │  │ Shared  │  │Separate │  │  Separate   │   │ │
│  │  │  Row    │  │ Schema  │  │Database │  │   Server    │   │ │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘   │ │
│  │                                                             │ │
│  │  Lower cost                             Higher isolation   │ │
│  │  Less isolation                         Higher cost        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Shared Database, Shared Schema

```
┌─────────────────────────────────────────────────────────────────┐
│              Row-Level Isolation (Tenant ID Column)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SCHEMA DESIGN                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE tenants (                                     │ │
│  │     id UUID PRIMARY KEY,                                   │ │
│  │     name VARCHAR(255),                                     │ │
│  │     plan VARCHAR(50),                                      │ │
│  │     created_at TIMESTAMP                                   │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ CREATE TABLE users (                                       │ │
│  │     id UUID PRIMARY KEY,                                   │ │
│  │     tenant_id UUID NOT NULL REFERENCES tenants(id),        │ │
│  │     email VARCHAR(255),                                    │ │
│  │     name VARCHAR(100),                                     │ │
│  │     UNIQUE (tenant_id, email)  -- unique within tenant    │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ CREATE TABLE projects (                                    │ │
│  │     id UUID PRIMARY KEY,                                   │ │
│  │     tenant_id UUID NOT NULL REFERENCES tenants(id),        │ │
│  │     name VARCHAR(255),                                     │ │
│  │     owner_id UUID REFERENCES users(id)                     │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Index for tenant queries                                │ │
│  │ CREATE INDEX idx_users_tenant ON users(tenant_id);         │ │
│  │ CREATE INDEX idx_projects_tenant ON projects(tenant_id);   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ROW-LEVEL SECURITY (PostgreSQL)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Enable RLS                                              │ │
│  │ ALTER TABLE users ENABLE ROW LEVEL SECURITY;               │ │
│  │ ALTER TABLE projects ENABLE ROW LEVEL SECURITY;            │ │
│  │                                                             │ │
│  │ -- Policy: users can only see their tenant's data         │ │
│  │ CREATE POLICY tenant_isolation ON users                    │ │
│  │ USING (tenant_id = current_setting('app.tenant_id')::uuid);│ │
│  │                                                             │ │
│  │ CREATE POLICY tenant_isolation ON projects                 │ │
│  │ USING (tenant_id = current_setting('app.tenant_id')::uuid);│ │
│  │                                                             │ │
│  │ -- Set tenant context per request                          │ │
│  │ SET app.tenant_id = 'abc-123-tenant-id';                   │ │
│  │ SELECT * FROM users; -- Only sees tenant's users          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PROS AND CONS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Lowest cost (shared everything)                         │ │
│  │ ✓ Simple deployment                                        │ │
│  │ ✓ Easy cross-tenant queries (admin)                       │ │
│  │                                                             │ │
│  │ ✗ Risk of data leaks (must always filter by tenant_id)   │ │
│  │ ✗ No per-tenant customization                             │ │
│  │ ✗ Noisy neighbor issues                                   │ │
│  │ ✗ Harder to meet strict compliance (HIPAA, etc.)         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Shared Database, Separate Schema

```
┌─────────────────────────────────────────────────────────────────┐
│              Schema-per-Tenant Isolation                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ARCHITECTURE                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                    Database                                │ │
│  │ ┌────────────────────────────────────────────────────────┐ │ │
│  │ │ public schema (shared)                                 │ │ │
│  │ │   tenants table                                        │ │ │
│  │ │   global_settings                                      │ │ │
│  │ └────────────────────────────────────────────────────────┘ │ │
│  │ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │ │
│  │ │ tenant_001  │ │ tenant_002  │ │ tenant_003  │          │ │
│  │ │  users      │ │  users      │ │  users      │          │ │
│  │ │  projects   │ │  projects   │ │  projects   │          │ │
│  │ │  ...        │ │  ...        │ │  ...        │          │ │
│  │ └─────────────┘ └─────────────┘ └─────────────┘          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  IMPLEMENTATION                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Create tenant schema                                    │ │
│  │ CREATE SCHEMA tenant_acme;                                 │ │
│  │                                                             │ │
│  │ -- Create tables in tenant schema                          │ │
│  │ CREATE TABLE tenant_acme.users (                           │ │
│  │     id UUID PRIMARY KEY,                                   │ │
│  │     email VARCHAR(255) UNIQUE,                             │ │
│  │     name VARCHAR(100)                                      │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Set search path for tenant                              │ │
│  │ SET search_path TO tenant_acme, public;                    │ │
│  │ SELECT * FROM users; -- Uses tenant_acme.users            │ │
│  │                                                             │ │
│  │ -- Or use fully qualified names                            │ │
│  │ SELECT * FROM tenant_acme.users;                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TENANT PROVISIONING                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Template for new tenants                                │ │
│  │ CREATE OR REPLACE FUNCTION create_tenant(tenant_name TEXT) │ │
│  │ RETURNS void AS $$                                         │ │
│  │ BEGIN                                                       │ │
│  │     EXECUTE format('CREATE SCHEMA %I', tenant_name);       │ │
│  │                                                             │ │
│  │     -- Create all tables from template                     │ │
│  │     EXECUTE format('CREATE TABLE %I.users (                │ │
│  │         id UUID PRIMARY KEY,                               │ │
│  │         email VARCHAR(255),                                │ │
│  │         name VARCHAR(100)                                  │ │
│  │     )', tenant_name);                                      │ │
│  │                                                             │ │
│  │     -- ... more tables                                     │ │
│  │ END;                                                        │ │
│  │ $$ LANGUAGE plpgsql;                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PROS AND CONS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Better isolation than row-level                         │ │
│  │ ✓ Per-tenant schema customization possible                │ │
│  │ ✓ Easier backup/restore per tenant                        │ │
│  │ ✓ No tenant_id in every query                             │ │
│  │                                                             │ │
│  │ ✗ Schema management complexity                            │ │
│  │ ✗ Migrations must apply to all schemas                    │ │
│  │ ✗ Connection pool per schema or dynamic routing           │ │
│  │ ✗ PostgreSQL: Many schemas can impact performance         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Separate Database per Tenant

```
┌─────────────────────────────────────────────────────────────────┐
│              Database-per-Tenant Isolation                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ARCHITECTURE                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │      ┌────────────────────────────────────────────────┐    │ │
│  │      │           Tenant Router / Proxy                │    │ │
│  │      └─────────────────┬──────────────────────────────┘    │ │
│  │                        │                                    │ │
│  │      ┌─────────────────┼─────────────────┐                 │ │
│  │      │                 │                 │                  │ │
│  │      ▼                 ▼                 ▼                  │ │
│  │ ┌─────────┐      ┌─────────┐      ┌─────────┐             │ │
│  │ │ DB:     │      │ DB:     │      │ DB:     │             │ │
│  │ │tenant_a │      │tenant_b │      │tenant_c │             │ │
│  │ └─────────┘      └─────────┘      └─────────┘             │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TENANT ROUTING                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Central registry (separate admin database)             │ │
│  │ CREATE TABLE tenant_registry (                             │ │
│  │     id UUID PRIMARY KEY,                                   │ │
│  │     subdomain VARCHAR(100) UNIQUE,                         │ │
│  │     database_host VARCHAR(255),                            │ │
│  │     database_name VARCHAR(100),                            │ │
│  │     database_user VARCHAR(100),                            │ │
│  │     status VARCHAR(50),                                    │ │
│  │     created_at TIMESTAMP                                   │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Application looks up connection at request time        │ │
│  │ tenant = TenantRegistry.find_by(subdomain: request.host)  │ │
│  │ connection = establish_connection(tenant.connection_info) │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SHARDING CONSIDERATIONS                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Distribution strategies:                                   │ │
│  │                                                             │ │
│  │ • Size-based: Large tenants get dedicated databases       │ │
│  │ • Region-based: Databases per geographic region           │ │
│  │ • Plan-based: Premium tenants get dedicated resources     │ │
│  │                                                             │ │
│  │ Example:                                                    │ │
│  │ ┌────────────────────────────────────────────────────┐    │ │
│  │ │ Free tier:    Shared database (row isolation)     │    │ │
│  │ │ Pro tier:     Shared database (schema isolation)  │    │ │
│  │ │ Enterprise:   Dedicated database                   │    │ │
│  │ └────────────────────────────────────────────────────┘    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PROS AND CONS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Complete isolation                                       │ │
│  │ ✓ Per-tenant performance tuning                           │ │
│  │ ✓ Easy compliance (data residency)                        │ │
│  │ ✓ Independent backup/restore                               │ │
│  │ ✓ Per-tenant scaling                                       │ │
│  │                                                             │ │
│  │ ✗ Higher cost (more database instances)                   │ │
│  │ ✗ Complex connection management                           │ │
│  │ ✗ Cross-tenant queries difficult                          │ │
│  │ ✗ More operational overhead                               │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Hybrid Approaches

```
┌─────────────────────────────────────────────────────────────────┐
│              Hybrid Multi-Tenancy                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  POOL + DEDICATED MODEL                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌────────────────────────────────────────────────────┐    │ │
│  │  │        Shared Pool (Small Tenants)                 │    │ │
│  │  │  ┌─────────────────────────────────────────────┐   │    │ │
│  │  │  │ tenant_a │ tenant_b │ tenant_c │ ...        │   │    │ │
│  │  │  └─────────────────────────────────────────────┘   │    │ │
│  │  └────────────────────────────────────────────────────┘    │ │
│  │                                                             │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │ │
│  │  │   Dedicated  │  │   Dedicated  │  │   Dedicated  │     │ │
│  │  │   tenant_x   │  │   tenant_y   │  │   tenant_z   │     │ │
│  │  │  (Enterprise)│  │  (Enterprise)│  │  (Enterprise)│     │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘     │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MIGRATION PATH                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Start all tenants in shared pool                       │ │
│  │ 2. Monitor resource usage per tenant                       │ │
│  │ 3. Automatically migrate large tenants to dedicated       │ │
│  │ 4. Allow enterprise customers to choose isolation level   │ │
│  │                                                             │ │
│  │ Trigger migration when:                                    │ │
│  │ • Tenant exceeds data threshold (e.g., 10GB)              │ │
│  │ • Tenant exceeds query threshold (e.g., 1000 QPS)         │ │
│  │ • Compliance requirement (e.g., healthcare tenant)        │ │
│  │ • Customer upgrades to enterprise plan                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BEST PRACTICES                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Abstract tenant resolution in application layer         │ │
│  │ • Use connection pooling per tenant or shard              │ │
│  │ • Implement tenant context middleware                     │ │
│  │ • Monitor per-tenant metrics                              │ │
│  │ • Plan for tenant migration between tiers                 │ │
│  │ • Consider data residency requirements early              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
