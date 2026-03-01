# Auditing and Compliance

## Database Auditing

```
┌─────────────────────────────────────────────────────────────────┐
│              Audit Logging                                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHAT TO AUDIT                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Authentication:                                             │ │
│  │ • Login attempts (success/failure)                         │ │
│  │ • Password changes                                          │ │
│  │ • Account lockouts                                          │ │
│  │                                                             │ │
│  │ Authorization:                                              │ │
│  │ • Permission changes                                        │ │
│  │ • Role assignments                                          │ │
│  │ • Access denials                                            │ │
│  │                                                             │ │
│  │ Data Access:                                                │ │
│  │ • SELECT on sensitive tables                               │ │
│  │ • INSERT, UPDATE, DELETE operations                        │ │
│  │ • Bulk data exports                                        │ │
│  │                                                             │ │
│  │ Schema Changes:                                             │ │
│  │ • CREATE, ALTER, DROP                                      │ │
│  │ • Index changes                                            │ │
│  │ • Permission grants                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL AUDIT LOGGING                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- postgresql.conf                                         │ │
│  │ log_statement = 'all'          # or 'ddl', 'mod'          │ │
│  │ log_connections = on                                       │ │
│  │ log_disconnections = on                                    │ │
│  │ log_duration = on                                          │ │
│  │                                                             │ │
│  │ -- pgAudit extension (more granular)                       │ │
│  │ CREATE EXTENSION pgaudit;                                  │ │
│  │                                                             │ │
│  │ -- Audit all reads on specific table                       │ │
│  │ ALTER TABLE sensitive_data ENABLE ROW LEVEL SECURITY;      │ │
│  │ SET pgaudit.log = 'read, write';                           │ │
│  │                                                             │ │
│  │ -- Per-role audit settings                                 │ │
│  │ ALTER ROLE analyst SET pgaudit.log = 'read';               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MYSQL AUDIT                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Enable general query log (not for production)          │ │
│  │ SET GLOBAL general_log = 'ON';                             │ │
│  │                                                             │ │
│  │ -- MySQL Enterprise Audit (commercial)                    │ │
│  │ INSTALL PLUGIN audit_log SONAME 'audit_log.so';            │ │
│  │                                                             │ │
│  │ -- MariaDB Audit Plugin (open source)                      │ │
│  │ INSTALL PLUGIN server_audit SONAME 'server_audit.so';      │ │
│  │ SET GLOBAL server_audit_logging = ON;                      │ │
│  │ SET GLOBAL server_audit_events = 'CONNECT,QUERY_DDL';      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Custom Audit Tables

```
┌─────────────────────────────────────────────────────────────────┐
│              Application-Level Audit Logging                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  AUDIT TABLE SCHEMA                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE audit_log (                                   │ │
│  │     id BIGSERIAL PRIMARY KEY,                              │ │
│  │     timestamp TIMESTAMPTZ DEFAULT NOW(),                   │ │
│  │     user_id BIGINT,                                        │ │
│  │     action VARCHAR(50),      -- INSERT, UPDATE, DELETE    │ │
│  │     table_name VARCHAR(100),                               │ │
│  │     record_id VARCHAR(100),                                │ │
│  │     old_values JSONB,                                      │ │
│  │     new_values JSONB,                                      │ │
│  │     ip_address INET,                                       │ │
│  │     user_agent TEXT                                        │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ CREATE INDEX idx_audit_timestamp ON audit_log(timestamp);  │ │
│  │ CREATE INDEX idx_audit_table ON audit_log(table_name);     │ │
│  │ CREATE INDEX idx_audit_user ON audit_log(user_id);         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TRIGGER-BASED AUDITING                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE OR REPLACE FUNCTION audit_trigger()                 │ │
│  │ RETURNS TRIGGER AS $$                                      │ │
│  │ BEGIN                                                       │ │
│  │     INSERT INTO audit_log (                                │ │
│  │         user_id, action, table_name, record_id,            │ │
│  │         old_values, new_values                             │ │
│  │     ) VALUES (                                              │ │
│  │         current_setting('app.user_id')::bigint,            │ │
│  │         TG_OP,                                              │ │
│  │         TG_TABLE_NAME,                                     │ │
│  │         COALESCE(NEW.id, OLD.id)::text,                    │ │
│  │         CASE WHEN TG_OP = 'DELETE' OR TG_OP = 'UPDATE'    │ │
│  │              THEN to_jsonb(OLD) END,                       │ │
│  │         CASE WHEN TG_OP = 'INSERT' OR TG_OP = 'UPDATE'    │ │
│  │              THEN to_jsonb(NEW) END                        │ │
│  │     );                                                      │ │
│  │     RETURN COALESCE(NEW, OLD);                             │ │
│  │ END;                                                        │ │
│  │ $$ LANGUAGE plpgsql;                                       │ │
│  │                                                             │ │
│  │ CREATE TRIGGER customers_audit                             │ │
│  │ AFTER INSERT OR UPDATE OR DELETE ON customers              │ │
│  │ FOR EACH ROW EXECUTE FUNCTION audit_trigger();             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Compliance Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│              Common Compliance Standards                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GDPR (General Data Protection Regulation)                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Requirements:                                               │ │
│  │ • Right to access personal data                           │ │
│  │ • Right to erasure ("right to be forgotten")              │ │
│  │ • Data portability                                         │ │
│  │ • Breach notification within 72 hours                      │ │
│  │ • Privacy by design                                        │ │
│  │                                                             │ │
│  │ Database implications:                                      │ │
│  │ • Implement data export functionality                      │ │
│  │ • Soft delete or proper cascade delete                    │ │
│  │ • Audit logs of personal data access                      │ │
│  │ • Data residency (store in EU)                            │ │
│  │ • Encryption of personal data                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PCI-DSS (Payment Card Industry)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Requirements:                                               │ │
│  │ • Encrypt cardholder data at rest                          │ │
│  │ • Encrypt in transit                                       │ │
│  │ • Restrict access to cardholder data                      │ │
│  │ • Track all access to cardholder data                     │ │
│  │ • Mask PAN when displayed                                  │ │
│  │                                                             │ │
│  │ Implementation:                                             │ │
│  │ • Never store CVV                                          │ │
│  │ • Tokenize card numbers                                    │ │
│  │ • Column-level encryption                                  │ │
│  │ • Comprehensive audit logging                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  HIPAA (Healthcare)                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Requirements:                                               │ │
│  │ • Protect PHI (Protected Health Information)              │ │
│  │ • Access controls and audit trails                        │ │
│  │ • Encryption of PHI                                        │ │
│  │ • Business Associate Agreements                           │ │
│  │                                                             │ │
│  │ Implementation:                                             │ │
│  │ • Role-based access control                                │ │
│  │ • Detailed audit logging of PHI access                    │ │
│  │ • Encryption at rest and in transit                       │ │
│  │ • Regular access reviews                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SOC 2                                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Trust Principles:                                           │ │
│  │ • Security                                                  │ │
│  │ • Availability                                              │ │
│  │ • Processing integrity                                     │ │
│  │ • Confidentiality                                          │ │
│  │ • Privacy                                                   │ │
│  │                                                             │ │
│  │ Database controls:                                          │ │
│  │ • Access controls documented                               │ │
│  │ • Change management procedures                             │ │
│  │ • Monitoring and alerting                                  │ │
│  │ • Backup and recovery tested                               │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Data Masking

```
┌─────────────────────────────────────────────────────────────────┐
│              Data Masking Techniques                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STATIC MASKING (for non-production)                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Create masked copy for dev/test                         │ │
│  │ CREATE TABLE dev_customers AS                              │ │
│  │ SELECT                                                      │ │
│  │     id,                                                     │ │
│  │     CONCAT('user_', id) AS email,                          │ │
│  │     'XXXX-XXXX-XXXX-' || RIGHT(ssn, 4) AS ssn,            │ │
│  │     CONCAT(LEFT(phone, 3), '-XXX-XXXX') AS phone           │ │
│  │ FROM customers;                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DYNAMIC MASKING (at query time)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- PostgreSQL view with masking                            │ │
│  │ CREATE VIEW masked_customers AS                            │ │
│  │ SELECT                                                      │ │
│  │     id,                                                     │ │
│  │     name,                                                   │ │
│  │     CASE                                                    │ │
│  │         WHEN current_user = 'admin'                        │ │
│  │         THEN email                                          │ │
│  │         ELSE REGEXP_REPLACE(email, '.+@', '***@')          │ │
│  │     END AS email,                                           │ │
│  │     CASE                                                    │ │
│  │         WHEN current_user = 'admin'                        │ │
│  │         THEN ssn                                            │ │
│  │         ELSE 'XXX-XX-' || RIGHT(ssn, 4)                    │ │
│  │     END AS ssn                                              │ │
│  │ FROM customers;                                             │ │
│  │                                                             │ │
│  │ -- Grant access to view, not table                         │ │
│  │ REVOKE SELECT ON customers FROM analyst;                   │ │
│  │ GRANT SELECT ON masked_customers TO analyst;               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MASKING TECHNIQUES                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Technique        │ Example                                 │ │
│  │ ─────────────────┼───────────────────────────────────────  │ │
│  │ Substitution     │ Real name → Fake name                  │ │
│  │ Shuffling        │ Swap values between records            │ │
│  │ Nulling          │ Replace with NULL                      │ │
│  │ Truncation       │ 123-45-6789 → XXX-XX-6789             │ │
│  │ Encryption       │ Original → Encrypted value            │ │
│  │ Tokenization     │ 4111111111111111 → tok_abc123         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
