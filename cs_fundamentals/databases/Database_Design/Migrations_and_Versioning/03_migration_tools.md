# Migration Tools

## Popular Migration Tools

```
┌─────────────────────────────────────────────────────────────────┐
│              Migration Tool Comparison                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FLYWAY                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Language: Java (runs anywhere)                             │ │
│  │ File format: Plain SQL or Java                            │ │
│  │ Naming: V1__description.sql                                │ │
│  │                                                             │ │
│  │ Commands:                                                   │ │
│  │ $ flyway migrate    # Apply pending migrations            │ │
│  │ $ flyway info       # Show status                         │ │
│  │ $ flyway validate   # Verify checksums                    │ │
│  │ $ flyway repair     # Fix metadata table                  │ │
│  │                                                             │ │
│  │ Best for: Java projects, enterprise                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  LIQUIBASE                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Language: Java                                              │ │
│  │ File format: XML, YAML, JSON, or SQL                      │ │
│  │                                                             │ │
│  │ Example (YAML):                                             │ │
│  │ databaseChangeLog:                                          │ │
│  │   - changeSet:                                              │ │
│  │       id: 1                                                 │ │
│  │       author: dev                                           │ │
│  │       changes:                                              │ │
│  │         - createTable:                                      │ │
│  │             tableName: users                                │ │
│  │             columns:                                        │ │
│  │               - column:                                     │ │
│  │                   name: id                                  │ │
│  │                   type: bigint                              │ │
│  │                                                             │ │
│  │ Best for: Complex workflows, rollback support             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ALEMBIC (Python)                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Works with: SQLAlchemy                                     │ │
│  │ File format: Python                                        │ │
│  │                                                             │ │
│  │ # alembic/versions/abc123_add_users.py                    │ │
│  │ def upgrade():                                              │ │
│  │     op.create_table('users',                               │ │
│  │         sa.Column('id', sa.Integer, primary_key=True),     │ │
│  │         sa.Column('email', sa.String(255))                 │ │
│  │     )                                                       │ │
│  │                                                             │ │
│  │ def downgrade():                                            │ │
│  │     op.drop_table('users')                                 │ │
│  │                                                             │ │
│  │ Commands:                                                   │ │
│  │ $ alembic revision -m "add users"                         │ │
│  │ $ alembic upgrade head                                     │ │
│  │ $ alembic downgrade -1                                     │ │
│  │                                                             │ │
│  │ Best for: Python/SQLAlchemy projects                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PRISMA MIGRATE                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Works with: Node.js, TypeScript                           │ │
│  │ Approach: Schema-first                                     │ │
│  │                                                             │ │
│  │ // schema.prisma                                           │ │
│  │ model User {                                                │ │
│  │   id    Int     @id @default(autoincrement())             │ │
│  │   email String  @unique                                    │ │
│  │   name  String?                                            │ │
│  │ }                                                           │ │
│  │                                                             │ │
│  │ Commands:                                                   │ │
│  │ $ prisma migrate dev     # Create and apply               │ │
│  │ $ prisma migrate deploy  # Apply in production            │ │
│  │                                                             │ │
│  │ Best for: TypeScript projects, type safety                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  GOOSE (Go)                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ File format: SQL or Go                                     │ │
│  │                                                             │ │
│  │ -- +goose Up                                               │ │
│  │ CREATE TABLE users (id SERIAL PRIMARY KEY);                │ │
│  │                                                             │ │
│  │ -- +goose Down                                             │ │
│  │ DROP TABLE users;                                          │ │
│  │                                                             │ │
│  │ $ goose up                                                  │ │
│  │ $ goose down                                                │ │
│  │                                                             │ │
│  │ Best for: Go projects, simple SQL migrations              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## ORM Migration Systems

```
┌─────────────────────────────────────────────────────────────────┐
│              Framework-Specific Tools                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RAILS ACTIVE RECORD                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ $ rails generate migration CreateUsers                     │ │
│  │                                                             │ │
│  │ class CreateUsers < ActiveRecord::Migration[7.0]           │ │
│  │   def change                                                │ │
│  │     create_table :users do |t|                             │ │
│  │       t.string :email, null: false                         │ │
│  │       t.string :name                                        │ │
│  │       t.timestamps                                          │ │
│  │     end                                                     │ │
│  │     add_index :users, :email, unique: true                 │ │
│  │   end                                                       │ │
│  │ end                                                         │ │
│  │                                                             │ │
│  │ $ rails db:migrate                                          │ │
│  │ $ rails db:rollback                                         │ │
│  │ $ rails db:migrate:status                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DJANGO                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # models.py                                                 │ │
│  │ class User(models.Model):                                   │ │
│  │     email = models.EmailField(unique=True)                 │ │
│  │     name = models.CharField(max_length=100)                │ │
│  │                                                             │ │
│  │ $ python manage.py makemigrations                          │ │
│  │ $ python manage.py migrate                                  │ │
│  │ $ python manage.py showmigrations                          │ │
│  │                                                             │ │
│  │ Features:                                                   │ │
│  │ • Auto-generates from model changes                        │ │
│  │ • Dependency tracking                                      │ │
│  │ • Squashing old migrations                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ENTITY FRAMEWORK (C#)                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ $ dotnet ef migrations add CreateUsers                     │ │
│  │ $ dotnet ef database update                                 │ │
│  │                                                             │ │
│  │ Features:                                                   │ │
│  │ • Scaffolds from DbContext changes                         │ │
│  │ • Generates SQL scripts                                    │ │
│  │ • Bundles for deployment                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## CI/CD Integration

```
┌─────────────────────────────────────────────────────────────────┐
│              Automating Migrations                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GITHUB ACTIONS EXAMPLE                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ name: Database Migration                                   │ │
│  │                                                             │ │
│  │ on:                                                         │ │
│  │   push:                                                     │ │
│  │     branches: [main]                                        │ │
│  │     paths:                                                  │ │
│  │       - 'db/migrations/**'                                 │ │
│  │                                                             │ │
│  │ jobs:                                                       │ │
│  │   migrate:                                                  │ │
│  │     runs-on: ubuntu-latest                                 │ │
│  │     steps:                                                  │ │
│  │       - uses: actions/checkout@v3                          │ │
│  │                                                             │ │
│  │       - name: Run migrations                               │ │
│  │         env:                                                │ │
│  │           DATABASE_URL: ${{ secrets.DATABASE_URL }}        │ │
│  │         run: |                                             │ │
│  │           flyway -url=$DATABASE_URL migrate                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  KUBERNETES JOB                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ apiVersion: batch/v1                                       │ │
│  │ kind: Job                                                   │ │
│  │ metadata:                                                   │ │
│  │   name: db-migration                                        │ │
│  │ spec:                                                       │ │
│  │   template:                                                 │ │
│  │     spec:                                                   │ │
│  │       containers:                                           │ │
│  │       - name: migrate                                       │ │
│  │         image: flyway/flyway                               │ │
│  │         args: ["migrate"]                                  │ │
│  │         env:                                                │ │
│  │         - name: FLYWAY_URL                                 │ │
│  │           valueFrom:                                        │ │
│  │             secretKeyRef:                                   │ │
│  │               name: db-secrets                             │ │
│  │               key: url                                      │ │
│  │       restartPolicy: Never                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BEST PRACTICES                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Run migrations before deploying new code                │ │
│  │ • Use separate credentials for migrations                  │ │
│  │ • Log migration output                                     │ │
│  │ • Alert on migration failures                              │ │
│  │ • Test migrations in staging first                        │ │
│  │ • Have rollback plan ready                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
