# Authentication and Authorization

## Authentication Methods

```
┌─────────────────────────────────────────────────────────────────┐
│              Database Authentication Methods                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PASSWORD AUTHENTICATION                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- PostgreSQL: Create user with password                  │ │
│  │ CREATE USER app_user WITH PASSWORD 'secure_password';     │ │
│  │                                                             │ │
│  │ -- MySQL: Create user with password                        │ │
│  │ CREATE USER 'app_user'@'%' IDENTIFIED BY 'password';       │ │
│  │                                                             │ │
│  │ Best practices:                                             │ │
│  │ • Minimum 16 characters                                    │ │
│  │ • Mix of upper, lower, numbers, symbols                   │ │
│  │ • Rotate credentials regularly                            │ │
│  │ • Store in secrets manager (not code)                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CERTIFICATE AUTHENTICATION                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- PostgreSQL pg_hba.conf                                  │ │
│  │ hostssl all all 0.0.0.0/0 cert clientcert=verify-full     │ │
│  │                                                             │ │
│  │ -- Connection with certificate                             │ │
│  │ psql "host=db.example.com dbname=mydb \                    │ │
│  │       sslmode=verify-full \                                │ │
│  │       sslcert=client.crt \                                 │ │
│  │       sslkey=client.key \                                  │ │
│  │       sslrootcert=ca.crt"                                  │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ • No passwords to manage                                   │ │
│  │ • Stronger authentication                                  │ │
│  │ • Certificate revocation possible                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  IAM AUTHENTICATION (Cloud)                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ AWS RDS IAM Authentication:                                │ │
│  │                                                             │ │
│  │ 1. Enable IAM auth on RDS instance                        │ │
│  │ 2. Create database user mapped to IAM                     │ │
│  │    CREATE USER app_user WITH LOGIN;                        │ │
│  │    GRANT rds_iam TO app_user;                              │ │
│  │                                                             │ │
│  │ 3. Connect using IAM credentials                          │ │
│  │    token = rds.generate_db_auth_token(...)                │ │
│  │    connection = psycopg2.connect(                          │ │
│  │        host=host,                                          │ │
│  │        user=user,                                          │ │
│  │        password=token,                                     │ │
│  │        database=database                                   │ │
│  │    )                                                        │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ • Centralized access management                            │ │
│  │ • Short-lived tokens                                       │ │
│  │ • No password storage                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Role-Based Access Control

```
┌─────────────────────────────────────────────────────────────────┐
│              RBAC Implementation                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ROLE HIERARCHY                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │                    ┌──────────────┐                        │ │
│  │                    │   db_admin   │                        │ │
│  │                    │ (superuser)  │                        │ │
│  │                    └──────┬───────┘                        │ │
│  │                           │                                 │ │
│  │          ┌────────────────┼────────────────┐               │ │
│  │          │                │                │                │ │
│  │          ▼                ▼                ▼                │ │
│  │    ┌──────────┐    ┌──────────┐    ┌──────────────┐       │ │
│  │    │ db_write │    │ db_read  │    │ db_analytics │       │ │
│  │    │  (CRUD)  │    │ (SELECT) │    │ (read-only)  │       │ │
│  │    └──────────┘    └──────────┘    └──────────────┘       │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL ROLE SETUP                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Create roles                                            │ │
│  │ CREATE ROLE db_read NOLOGIN;                               │ │
│  │ CREATE ROLE db_write NOLOGIN;                              │ │
│  │ CREATE ROLE db_admin NOLOGIN;                              │ │
│  │                                                             │ │
│  │ -- Grant permissions to roles                              │ │
│  │ GRANT SELECT ON ALL TABLES IN SCHEMA public TO db_read;    │ │
│  │ GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES         │ │
│  │     IN SCHEMA public TO db_write;                          │ │
│  │ GRANT ALL PRIVILEGES ON ALL TABLES                         │ │
│  │     IN SCHEMA public TO db_admin;                          │ │
│  │                                                             │ │
│  │ -- Set default privileges for future tables                │ │
│  │ ALTER DEFAULT PRIVILEGES IN SCHEMA public                  │ │
│  │     GRANT SELECT ON TABLES TO db_read;                     │ │
│  │                                                             │ │
│  │ -- Create users and assign roles                           │ │
│  │ CREATE USER app_service WITH PASSWORD 'xxx';               │ │
│  │ GRANT db_write TO app_service;                             │ │
│  │                                                             │ │
│  │ CREATE USER analyst WITH PASSWORD 'xxx';                   │ │
│  │ GRANT db_read TO analyst;                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MYSQL ROLE SETUP                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Create roles (MySQL 8.0+)                               │ │
│  │ CREATE ROLE 'app_read', 'app_write';                       │ │
│  │                                                             │ │
│  │ -- Grant privileges to roles                               │ │
│  │ GRANT SELECT ON mydb.* TO 'app_read';                      │ │
│  │ GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.*             │ │
│  │     TO 'app_write';                                        │ │
│  │                                                             │ │
│  │ -- Assign roles to users                                   │ │
│  │ GRANT 'app_write' TO 'app_user'@'%';                       │ │
│  │ SET DEFAULT ROLE 'app_write' TO 'app_user'@'%';            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Row-Level Security

```
┌─────────────────────────────────────────────────────────────────┐
│              Row-Level Security (RLS)                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  POSTGRESQL RLS                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Enable RLS on table                                     │ │
│  │ ALTER TABLE orders ENABLE ROW LEVEL SECURITY;              │ │
│  │                                                             │ │
│  │ -- Policy: Users see only their own orders                │ │
│  │ CREATE POLICY user_orders ON orders                        │ │
│  │     FOR ALL                                                 │ │
│  │     USING (user_id = current_setting('app.user_id')::int); │ │
│  │                                                             │ │
│  │ -- Policy: Admins see all orders                          │ │
│  │ CREATE POLICY admin_all_orders ON orders                   │ │
│  │     FOR ALL                                                 │ │
│  │     TO admin_role                                          │ │
│  │     USING (true);                                          │ │
│  │                                                             │ │
│  │ -- Set user context in application                        │ │
│  │ SET app.user_id = '123';                                   │ │
│  │ SELECT * FROM orders; -- Only sees user 123's orders      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MULTI-TENANT RLS                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Tenant isolation policy                                 │ │
│  │ CREATE POLICY tenant_isolation ON customers                │ │
│  │     USING (tenant_id = current_setting('app.tenant_id'));  │ │
│  │                                                             │ │
│  │ -- Combined with column-level permissions                  │ │
│  │ CREATE POLICY customer_support ON customers                │ │
│  │     FOR SELECT                                              │ │
│  │     TO support_role                                        │ │
│  │     USING (tenant_id = current_setting('app.tenant_id'))   │ │
│  │     -- Can only see specific columns via view             │ │
│  │     ;                                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Principle of Least Privilege

```
┌─────────────────────────────────────────────────────────────────┐
│              Least Privilege Examples                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  APPLICATION SERVICE ACCOUNTS                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Bad:  Application uses db superuser                       │ │
│  │ Good: Application has specific permissions                 │ │
│  │                                                             │ │
│  │ -- Web application service account                        │ │
│  │ CREATE USER webapp WITH PASSWORD 'xxx';                    │ │
│  │ GRANT SELECT, INSERT, UPDATE ON users TO webapp;           │ │
│  │ GRANT SELECT, INSERT ON orders TO webapp;                  │ │
│  │ GRANT SELECT ON products TO webapp;                        │ │
│  │ -- No DELETE, no schema changes                           │ │
│  │                                                             │ │
│  │ -- Background job service account                          │ │
│  │ CREATE USER jobs WITH PASSWORD 'xxx';                      │ │
│  │ GRANT SELECT, UPDATE ON orders TO jobs;                    │ │
│  │ GRANT INSERT ON notifications TO jobs;                     │ │
│  │ -- Only tables it needs                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SEPARATION OF DUTIES                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Account Type      │ Permissions                           │ │
│  │ ──────────────────┼─────────────────────────────────────  │ │
│  │ app_read          │ SELECT only                           │ │
│  │ app_write         │ SELECT, INSERT, UPDATE, DELETE        │ │
│  │ app_migrate       │ CREATE, ALTER, DROP (schema only)     │ │
│  │ app_admin         │ GRANT, user management                │ │
│  │ backup_user       │ SELECT, pg_dump privileges            │ │
│  │ monitoring        │ pg_stat views only                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
