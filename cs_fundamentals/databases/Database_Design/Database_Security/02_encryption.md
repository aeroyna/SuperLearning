# Encryption

## Encryption in Transit

```
┌─────────────────────────────────────────────────────────────────┐
│              TLS/SSL for Database Connections                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHY ENCRYPT IN TRANSIT                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Without TLS:                                               │ │
│  │ [App] ──── plaintext queries ────► [Database]             │ │
│  │       ◄─── plaintext data ────────                         │ │
│  │                │                                            │ │
│  │                ▼ Attacker can see/modify                   │ │
│  │                                                             │ │
│  │ With TLS:                                                   │ │
│  │ [App] ══════ encrypted ══════════► [Database]             │ │
│  │       ◄══════ encrypted ══════════                         │ │
│  │                │                                            │ │
│  │                ✗ Attacker sees gibberish                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL TLS CONFIGURATION                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Server: postgresql.conf                                 │ │
│  │ ssl = on                                                    │ │
│  │ ssl_cert_file = '/path/to/server.crt'                      │ │
│  │ ssl_key_file = '/path/to/server.key'                       │ │
│  │ ssl_ca_file = '/path/to/ca.crt'                            │ │
│  │                                                             │ │
│  │ -- pg_hba.conf: Require SSL                                │ │
│  │ hostssl all all 0.0.0.0/0 md5                              │ │
│  │                                                             │ │
│  │ -- Client connection                                       │ │
│  │ psql "host=db.example.com sslmode=verify-full \            │ │
│  │       sslrootcert=ca.crt dbname=mydb"                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SSL MODES                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Mode            Security   Use Case                       │ │
│  │ ───────────────────────────────────────────────────────── │ │
│  │ disable         None       Never (testing only)           │ │
│  │ allow           Low        Fallback if SSL unavailable    │ │
│  │ prefer          Medium     Use SSL if available           │ │
│  │ require         High       SSL required, no cert verify   │ │
│  │ verify-ca       Higher     Verify CA certificate          │ │
│  │ verify-full     Highest    Verify CA + hostname           │ │
│  │                                                             │ │
│  │ Recommendation: Always use verify-full in production      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MYSQL TLS CONFIGURATION                                        │ │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Server: my.cnf                                          │ │
│  │ [mysqld]                                                    │ │
│  │ ssl-ca=/path/to/ca.crt                                     │ │
│  │ ssl-cert=/path/to/server.crt                               │ │
│  │ ssl-key=/path/to/server.key                                │ │
│  │ require_secure_transport=ON                                │ │
│  │                                                             │ │
│  │ -- Require SSL for user                                    │ │
│  │ ALTER USER 'app'@'%' REQUIRE SSL;                          │ │
│  │                                                             │ │
│  │ -- Client connection                                       │ │
│  │ mysql --ssl-ca=ca.crt --ssl-mode=VERIFY_IDENTITY           │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Encryption at Rest

```
┌─────────────────────────────────────────────────────────────────┐
│              Data-at-Rest Encryption                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRANSPARENT DATA ENCRYPTION (TDE)                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Database encrypts/decrypts data transparently              │ │
│  │                                                             │ │
│  │ ┌──────────┐    ┌──────────────┐    ┌────────────────┐    │ │
│  │ │   App    │───▶│   Database   │───▶│  Disk (AES)    │    │ │
│  │ │(plaintext│    │  Engine      │    │  Encrypted     │    │ │
│  │ │  queries)│    │(encrypt/     │    │  data files    │    │ │
│  │ └──────────┘    │ decrypt)     │    └────────────────┘    │ │
│  │                 └──────────────┘                           │ │
│  │                        │                                    │ │
│  │                        ▼                                    │ │
│  │                 ┌──────────────┐                           │ │
│  │                 │  Key Mgmt    │                           │ │
│  │                 │  (KMS/HSM)   │                           │ │
│  │                 └──────────────┘                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CLOUD PROVIDER ENCRYPTION                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ AWS RDS:                                                    │ │
│  │ • Enable encryption at creation time                       │ │
│  │ • Uses AWS KMS for key management                          │ │
│  │ • AES-256 encryption                                       │ │
│  │ • Includes backups and replicas                            │ │
│  │                                                             │ │
│  │ GCP Cloud SQL:                                              │ │
│  │ • Encrypted by default                                     │ │
│  │ • Customer-managed keys (CMEK) optional                    │ │
│  │                                                             │ │
│  │ Azure SQL:                                                  │ │
│  │ • TDE enabled by default                                   │ │
│  │ • Azure Key Vault integration                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  FILE SYSTEM ENCRYPTION                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Alternative: Encrypt at filesystem/volume level            │ │
│  │                                                             │ │
│  │ • Linux: LUKS/dm-crypt                                     │ │
│  │ • AWS: EBS encryption                                      │ │
│  │ • GCP: Persistent disk encryption                          │ │
│  │                                                             │ │
│  │ Pros: Works with any database                              │ │
│  │ Cons: Less granular than TDE                               │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Column-Level Encryption

```
┌─────────────────────────────────────────────────────────────────┐
│              Application-Level Encryption                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHEN TO USE                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Specific sensitive columns (SSN, credit card)           │ │
│  │ • Compliance requirements (PCI-DSS, HIPAA)                 │ │
│  │ • Protection from database admins                          │ │
│  │ • Multi-tenant data isolation                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POSTGRESQL pgcrypto                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE EXTENSION pgcrypto;                                  │ │
│  │                                                             │ │
│  │ -- Encrypt data                                            │ │
│  │ INSERT INTO users (email, ssn_encrypted)                   │ │
│  │ VALUES (                                                    │ │
│  │     'user@example.com',                                    │ │
│  │     pgp_sym_encrypt('123-45-6789', 'encryption_key')       │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Decrypt data                                            │ │
│  │ SELECT email,                                               │ │
│  │        pgp_sym_decrypt(ssn_encrypted, 'encryption_key')    │ │
│  │        AS ssn                                               │ │
│  │ FROM users;                                                 │ │
│  │                                                             │ │
│  │ Note: Key should come from secure source, not hardcoded   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  APPLICATION-SIDE ENCRYPTION                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Python example with cryptography library                │ │
│  │ from cryptography.fernet import Fernet                     │ │
│  │                                                             │ │
│  │ # Key from secure storage (KMS, Vault)                    │ │
│  │ key = get_encryption_key_from_vault()                      │ │
│  │ cipher = Fernet(key)                                        │ │
│  │                                                             │ │
│  │ # Encrypt before storing                                   │ │
│  │ encrypted_ssn = cipher.encrypt(b"123-45-6789")             │ │
│  │ db.execute("INSERT INTO users (ssn) VALUES (?)",           │ │
│  │            [encrypted_ssn])                                 │ │
│  │                                                             │ │
│  │ # Decrypt after reading                                    │ │
│  │ row = db.execute("SELECT ssn FROM users WHERE id=?")       │ │
│  │ ssn = cipher.decrypt(row.ssn)                              │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ • Database never sees plaintext                            │ │
│  │ • Works with any database                                  │ │
│  │ • Per-tenant keys possible                                 │ │
│  │                                                             │ │
│  │ Drawbacks:                                                  │ │
│  │ • Cannot query encrypted data                              │ │
│  │ • Key management complexity                                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Key Management

```
┌─────────────────────────────────────────────────────────────────┐
│              Key Management Best Practices                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  KEY MANAGEMENT SERVICES                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ AWS KMS:                                                    │ │
│  │ • Managed key storage                                      │ │
│  │ • Automatic key rotation                                   │ │
│  │ • IAM integration                                          │ │
│  │ • Audit logging                                            │ │
│  │                                                             │ │
│  │ HashiCorp Vault:                                           │ │
│  │ • Self-hosted or cloud                                     │ │
│  │ • Dynamic secrets                                          │ │
│  │ • Encryption as a service                                  │ │
│  │ • Fine-grained access control                              │ │
│  │                                                             │ │
│  │ Azure Key Vault / GCP Cloud KMS:                           │ │
│  │ • Similar to AWS KMS                                       │ │
│  │ • Native cloud integration                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  KEY ROTATION                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Strategy for rotating encryption keys:                     │ │
│  │                                                             │ │
│  │ 1. Generate new key (keep old key active)                 │ │
│  │ 2. New data encrypted with new key                        │ │
│  │ 3. Background job re-encrypts old data                    │ │
│  │ 4. Retire old key after all data migrated                 │ │
│  │                                                             │ │
│  │ -- Track which key version encrypted each row             │ │
│  │ CREATE TABLE users (                                       │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     ssn_encrypted BYTEA,                                   │ │
│  │     encryption_key_version INT DEFAULT 1                   │ │
│  │ );                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NEVER DO                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✗ Store keys in source code                               │ │
│  │ ✗ Store keys in same database as data                    │ │
│  │ ✗ Use weak/predictable keys                               │ │
│  │ ✗ Reuse keys across environments                          │ │
│  │ ✗ Never rotate keys                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
