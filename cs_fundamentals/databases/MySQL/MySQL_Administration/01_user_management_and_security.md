# User Management and Security

## Learning Objectives
- Master MySQL user and privilege management
- Implement authentication best practices
- Secure MySQL installations
- Audit and monitor access

---

## 1. User Accounts

### Creating Users

```sql
-- Basic user creation
CREATE USER 'username'@'hostname' IDENTIFIED BY 'password';

-- Examples:
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'SecurePass123!';
CREATE USER 'admin'@'192.168.1.%' IDENTIFIED BY 'AdminPass456!';
CREATE USER 'readonly'@'%' IDENTIFIED BY 'ReadOnly789!';

-- With specific authentication plugin
CREATE USER 'user'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'password';

-- With password expiration
CREATE USER 'temp_user'@'localhost' IDENTIFIED BY 'TempPass!'
    PASSWORD EXPIRE INTERVAL 90 DAY;

-- Locked account (for later activation)
CREATE USER 'future_user'@'localhost' IDENTIFIED BY 'Pass!'
    ACCOUNT LOCK;
```

### Modifying Users

```sql
-- Change password
ALTER USER 'username'@'localhost' IDENTIFIED BY 'NewPassword123!';

-- Rename user
RENAME USER 'old_name'@'localhost' TO 'new_name'@'localhost';

-- Unlock account
ALTER USER 'username'@'localhost' ACCOUNT UNLOCK;

-- Expire password immediately
ALTER USER 'username'@'localhost' PASSWORD EXPIRE;

-- Remove password expiration
ALTER USER 'username'@'localhost' PASSWORD EXPIRE NEVER;

-- Set resource limits
ALTER USER 'username'@'localhost'
    WITH MAX_QUERIES_PER_HOUR 1000
         MAX_UPDATES_PER_HOUR 100
         MAX_CONNECTIONS_PER_HOUR 50
         MAX_USER_CONNECTIONS 5;
```

### Viewing Users

```sql
-- List all users
SELECT User, Host, plugin, authentication_string
FROM mysql.user;

-- Current user
SELECT USER(), CURRENT_USER();

-- User privileges
SHOW GRANTS FOR 'username'@'localhost';
SHOW GRANTS FOR CURRENT_USER;
```

### Deleting Users

```sql
-- Drop user
DROP USER 'username'@'localhost';

-- Drop if exists
DROP USER IF EXISTS 'username'@'localhost';
```

---

## 2. Privilege System

### Privilege Levels

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MySQL Privilege Hierarchy                         │
│                                                                      │
│  GLOBAL (*.*)                                                        │
│  ├── All databases and tables                                        │
│  ├── Administrative privileges                                       │
│  │                                                                   │
│  │   DATABASE (db.*)                                                 │
│  │   ├── All tables in specific database                            │
│  │   │                                                               │
│  │   │   TABLE (db.table)                                            │
│  │   │   ├── Specific table                                          │
│  │   │   │                                                           │
│  │   │   │   COLUMN (db.table.column)                                │
│  │   │   │   └── Specific columns                                    │
│  │   │   │                                                           │
│  │   │   ROUTINE (db.procedure/function)                             │
│  │   │   └── Stored procedures and functions                         │
│  │   │                                                               │
│  │   PROXY                                                           │
│  │   └── Proxy one user as another                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Common Privileges

```sql
-- Data privileges
GRANT SELECT ON db.* TO 'user'@'host';
GRANT INSERT ON db.* TO 'user'@'host';
GRANT UPDATE ON db.* TO 'user'@'host';
GRANT DELETE ON db.* TO 'user'@'host';

-- Structure privileges
GRANT CREATE ON db.* TO 'user'@'host';
GRANT ALTER ON db.* TO 'user'@'host';
GRANT DROP ON db.* TO 'user'@'host';
GRANT INDEX ON db.* TO 'user'@'host';

-- Administrative privileges
GRANT RELOAD ON *.* TO 'user'@'host';
GRANT SHUTDOWN ON *.* TO 'user'@'host';
GRANT PROCESS ON *.* TO 'user'@'host';
GRANT FILE ON *.* TO 'user'@'host';

-- All privileges
GRANT ALL PRIVILEGES ON db.* TO 'user'@'host';
```

### Granting Privileges

```sql
-- Database level
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'app_user'@'%';

-- Table level
GRANT SELECT ON myapp.users TO 'report_user'@'localhost';

-- Column level
GRANT SELECT (id, name, email) ON myapp.users TO 'limited_user'@'localhost';

-- With grant option (can grant to others)
GRANT SELECT ON myapp.* TO 'senior_user'@'localhost' WITH GRANT OPTION;

-- Apply changes
FLUSH PRIVILEGES;
```

### Revoking Privileges

```sql
-- Revoke specific privilege
REVOKE INSERT ON myapp.* FROM 'app_user'@'%';

-- Revoke all privileges
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'user'@'host';

-- Verify
SHOW GRANTS FOR 'user'@'host';
```

---

## 3. Roles (MySQL 8.0+)

### Creating and Using Roles

```sql
-- Create roles
CREATE ROLE 'app_read', 'app_write', 'app_admin';

-- Grant privileges to roles
GRANT SELECT ON myapp.* TO 'app_read';
GRANT INSERT, UPDATE, DELETE ON myapp.* TO 'app_write';
GRANT ALL PRIVILEGES ON myapp.* TO 'app_admin';

-- Assign roles to users
GRANT 'app_read' TO 'readonly_user'@'localhost';
GRANT 'app_read', 'app_write' TO 'app_user'@'localhost';
GRANT 'app_admin' TO 'admin_user'@'localhost';

-- Set default role
SET DEFAULT ROLE 'app_read' TO 'readonly_user'@'localhost';
SET DEFAULT ROLE ALL TO 'app_user'@'localhost';

-- Activate role in session
SET ROLE 'app_write';
SET ROLE ALL;
SET ROLE NONE;
```

### Viewing Roles

```sql
-- Show roles granted to user
SHOW GRANTS FOR 'app_user'@'localhost';

-- Show current active roles
SELECT CURRENT_ROLE();

-- Show all roles
SELECT User, Host FROM mysql.user WHERE account_locked = 'Y' AND password_expired = 'Y';
```

---

## 4. Authentication

### Authentication Plugins

```sql
-- View available plugins
SELECT PLUGIN_NAME, PLUGIN_STATUS FROM information_schema.PLUGINS
WHERE PLUGIN_TYPE = 'AUTHENTICATION';

-- Common plugins:
-- caching_sha2_password: Default in MySQL 8.0 (recommended)
-- mysql_native_password: Legacy, compatible
-- sha256_password: SHA-256 based
-- auth_socket: Unix socket authentication

-- Create user with specific plugin
CREATE USER 'user'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'password';

-- Change authentication plugin
ALTER USER 'user'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

### Password Policy

```sql
-- View password policy
SHOW VARIABLES LIKE 'validate_password%';

-- Configure policy
SET GLOBAL validate_password.policy = STRONG;  -- LOW, MEDIUM, STRONG
SET GLOBAL validate_password.length = 12;
SET GLOBAL validate_password.mixed_case_count = 1;
SET GLOBAL validate_password.number_count = 1;
SET GLOBAL validate_password.special_char_count = 1;

-- In my.cnf
[mysqld]
validate_password.policy = STRONG
validate_password.length = 12
```

### Password Expiration

```sql
-- Set default password lifetime
SET GLOBAL default_password_lifetime = 90;  -- days, 0 = never

-- Per-user setting
ALTER USER 'user'@'localhost' PASSWORD EXPIRE INTERVAL 60 DAY;
ALTER USER 'user'@'localhost' PASSWORD EXPIRE NEVER;

-- Check expiration
SELECT User, Host, password_expired, password_lifetime
FROM mysql.user;
```

---

## 5. Security Hardening

### Secure Installation

```bash
# Run security script
mysql_secure_installation

# This will:
# - Set root password
# - Remove anonymous users
# - Disable remote root login
# - Remove test database
# - Reload privilege tables
```

### Configuration Hardening

```ini
# /etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
# Bind to specific interface
bind-address = 127.0.0.1  # Or specific IP

# Disable local file loading
local_infile = 0

# Disable symbolic links
skip_symbolic_links = 1

# Disable SHOW DATABASES for non-privileged
skip_show_database

# Require SSL for connections
require_secure_transport = ON

# Log failed login attempts
log_error_verbosity = 3
```

### SSL/TLS Configuration

```sql
-- Check SSL status
SHOW VARIABLES LIKE '%ssl%';
SHOW STATUS LIKE 'Ssl%';

-- View current connection encryption
SHOW STATUS LIKE 'Ssl_cipher';

-- Require SSL for user
ALTER USER 'user'@'%' REQUIRE SSL;

-- Require specific SSL options
ALTER USER 'user'@'%' REQUIRE X509;
ALTER USER 'user'@'%' REQUIRE ISSUER '/C=US/ST=CA/O=Company/CN=ca';
```

### SSL Configuration

```ini
[mysqld]
ssl_ca = /etc/mysql/ca.pem
ssl_cert = /etc/mysql/server-cert.pem
ssl_key = /etc/mysql/server-key.pem
require_secure_transport = ON

[client]
ssl_ca = /etc/mysql/ca.pem
ssl_cert = /etc/mysql/client-cert.pem
ssl_key = /etc/mysql/client-key.pem
```

---

## 6. Auditing

### General Query Log

```sql
-- Enable general query log
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

-- Or to table
SET GLOBAL log_output = 'TABLE';

-- View log
SELECT * FROM mysql.general_log ORDER BY event_time DESC LIMIT 100;
```

### Enterprise Audit Plugin

```sql
-- Install audit plugin (Enterprise)
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

-- Configure
SET GLOBAL audit_log_policy = ALL;
SET GLOBAL audit_log_format = JSON;

-- Filter events
SET GLOBAL audit_log_include_accounts = 'app_user@%,admin@%';
```

### Connection Logging

```sql
-- Track connections via Performance Schema
SELECT
    USER,
    HOST,
    CURRENT_CONNECTIONS,
    TOTAL_CONNECTIONS
FROM performance_schema.accounts
WHERE USER IS NOT NULL;

-- Failed login attempts (error log)
-- [Warning] Access denied for user 'user'@'host'
```

---

## 7. Common Security Tasks

### Remove Anonymous Users

```sql
SELECT User, Host FROM mysql.user WHERE User = '';
DROP USER ''@'localhost';
DROP USER ''@'%';
```

### Restrict Root Access

```sql
-- Remove remote root access
DELETE FROM mysql.user WHERE User = 'root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
FLUSH PRIVILEGES;

-- Or rename root
RENAME USER 'root'@'localhost' TO 'admin'@'localhost';
```

### Application User Template

```sql
-- Create application user with minimal privileges
CREATE USER 'app_myservice'@'app-server.local'
    IDENTIFIED BY 'StrongPassword123!'
    PASSWORD EXPIRE INTERVAL 180 DAY
    FAILED_LOGIN_ATTEMPTS 3
    PASSWORD_LOCK_TIME 1;

GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'app_myservice'@'app-server.local';

-- Verify
SHOW GRANTS FOR 'app_myservice'@'app-server.local';
```

---

## Summary

| Task | Command |
|------|---------|
| Create user | `CREATE USER 'user'@'host' IDENTIFIED BY 'pass'` |
| Grant privilege | `GRANT SELECT ON db.* TO 'user'@'host'` |
| Revoke privilege | `REVOKE SELECT ON db.* FROM 'user'@'host'` |
| View grants | `SHOW GRANTS FOR 'user'@'host'` |
| Drop user | `DROP USER 'user'@'host'` |
| Require SSL | `ALTER USER 'user'@'host' REQUIRE SSL` |

---

## Further Reading

- MySQL Security documentation
- MySQL Enterprise Audit
- OWASP Database Security guidelines
