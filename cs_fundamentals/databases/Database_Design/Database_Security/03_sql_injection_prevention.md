# SQL Injection Prevention

## Understanding SQL Injection

```
┌─────────────────────────────────────────────────────────────────┐
│              SQL Injection Explained                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VULNERABLE CODE                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Python - VULNERABLE                                      │ │
│  │ username = request.form['username']                        │ │
│  │ query = f"SELECT * FROM users WHERE username = '{username}'"│ │
│  │ cursor.execute(query)                                       │ │
│  │                                                             │ │
│  │ # Normal input: "alice"                                    │ │
│  │ # Query: SELECT * FROM users WHERE username = 'alice'      │ │
│  │                                                             │ │
│  │ # Malicious input: "' OR '1'='1"                          │ │
│  │ # Query: SELECT * FROM users WHERE username = '' OR '1'='1'│ │
│  │ # Result: Returns ALL users                               │ │
│  │                                                             │ │
│  │ # Even worse: "'; DROP TABLE users; --"                   │ │
│  │ # Query: SELECT * FROM users WHERE username = '';          │ │
│  │ #        DROP TABLE users; --'                             │ │
│  │ # Result: Table deleted!                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ATTACK TYPES                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ In-band SQLi:                                               │ │
│  │ • Error-based: Trigger errors to extract data             │ │
│  │ • Union-based: Use UNION to retrieve other tables         │ │
│  │                                                             │ │
│  │ Blind SQLi:                                                 │ │
│  │ • Boolean-based: Infer data from true/false responses     │ │
│  │ • Time-based: Use delays to extract data bit by bit       │ │
│  │                                                             │ │
│  │ Out-of-band SQLi:                                          │ │
│  │ • DNS/HTTP requests to extract data                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Parameterized Queries

```
┌─────────────────────────────────────────────────────────────────┐
│              The Solution: Parameterized Queries                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PYTHON EXAMPLES                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # psycopg2 (PostgreSQL)                                    │ │
│  │ cursor.execute(                                             │ │
│  │     "SELECT * FROM users WHERE username = %s",             │ │
│  │     (username,)                                             │ │
│  │ )                                                           │ │
│  │                                                             │ │
│  │ # SQLAlchemy                                                │ │
│  │ from sqlalchemy import text                                 │ │
│  │ result = session.execute(                                   │ │
│  │     text("SELECT * FROM users WHERE username = :name"),    │ │
│  │     {"name": username}                                      │ │
│  │ )                                                           │ │
│  │                                                             │ │
│  │ # SQLAlchemy ORM (best)                                    │ │
│  │ user = session.query(User).filter(                         │ │
│  │     User.username == username                               │ │
│  │ ).first()                                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  JAVA EXAMPLES                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ // JDBC PreparedStatement                                  │ │
│  │ String sql = "SELECT * FROM users WHERE username = ?";     │ │
│  │ PreparedStatement stmt = connection.prepareStatement(sql); │ │
│  │ stmt.setString(1, username);                               │ │
│  │ ResultSet rs = stmt.executeQuery();                        │ │
│  │                                                             │ │
│  │ // JPA/Hibernate                                           │ │
│  │ Query query = em.createQuery(                              │ │
│  │     "SELECT u FROM User u WHERE u.username = :name"        │ │
│  │ );                                                          │ │
│  │ query.setParameter("name", username);                      │ │
│  │ User user = query.getSingleResult();                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NODE.JS EXAMPLES                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ // pg (node-postgres)                                      │ │
│  │ const result = await client.query(                         │ │
│  │   'SELECT * FROM users WHERE username = $1',               │ │
│  │   [username]                                                │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ // mysql2                                                   │ │
│  │ const [rows] = await connection.execute(                   │ │
│  │   'SELECT * FROM users WHERE username = ?',                │ │
│  │   [username]                                                │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ // Prisma ORM (best)                                       │ │
│  │ const user = await prisma.user.findUnique({                │ │
│  │   where: { username: username }                            │ │
│  │ });                                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  WHY IT WORKS                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Parameterized queries separate:                            │ │
│  │ • SQL structure (sent first)                               │ │
│  │ • Data values (sent separately)                            │ │
│  │                                                             │ │
│  │ The database knows what is code and what is data          │ │
│  │ User input is ALWAYS treated as data, never as code       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Additional Defenses

```
┌─────────────────────────────────────────────────────────────────┐
│              Defense in Depth                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INPUT VALIDATION                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Validate and sanitize input                              │ │
│  │                                                             │ │
│  │ # Allowlist approach (preferred)                           │ │
│  │ ALLOWED_SORT_COLUMNS = ['name', 'created_at', 'email']     │ │
│  │ if sort_column not in ALLOWED_SORT_COLUMNS:                │ │
│  │     raise ValueError("Invalid sort column")                 │ │
│  │                                                             │ │
│  │ # Type validation                                          │ │
│  │ user_id = int(request.args['id'])  # Fails if not integer │ │
│  │                                                             │ │
│  │ # Length limits                                            │ │
│  │ if len(username) > 50:                                     │ │
│  │     raise ValueError("Username too long")                   │ │
│  │                                                             │ │
│  │ Note: Validation is secondary defense, not primary         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  LEAST PRIVILEGE                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Limit damage if injection succeeds:                        │ │
│  │                                                             │ │
│  │ • App user can't DROP tables                               │ │
│  │ • App user can't access other databases                   │ │
│  │ • Read-only endpoints use read-only accounts              │ │
│  │                                                             │ │
│  │ CREATE USER webapp WITH PASSWORD 'xxx';                    │ │
│  │ GRANT SELECT, INSERT, UPDATE ON app_tables TO webapp;      │ │
│  │ -- No DELETE, no DDL privileges                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ERROR HANDLING                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Don't expose database errors to users                   │ │
│  │                                                             │ │
│  │ # Bad - reveals database structure                         │ │
│  │ try:                                                        │ │
│  │     result = db.execute(query)                             │ │
│  │ except Exception as e:                                      │ │
│  │     return str(e)  # "column 'xyz' does not exist"        │ │
│  │                                                             │ │
│  │ # Good - generic error message                             │ │
│  │ try:                                                        │ │
│  │     result = db.execute(query)                             │ │
│  │ except Exception as e:                                      │ │
│  │     logger.error(f"Database error: {e}")                   │ │
│  │     return "An error occurred", 500                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  WEB APPLICATION FIREWALL                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • AWS WAF                                                   │ │
│  │ • Cloudflare WAF                                           │ │
│  │ • ModSecurity                                               │ │
│  │                                                             │ │
│  │ Can detect and block common SQLi patterns                  │ │
│  │ Should not be only defense (can be bypassed)              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Testing for SQL Injection

```
┌─────────────────────────────────────────────────────────────────┐
│              Security Testing                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  AUTOMATED TOOLS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ SQLMap (open source):                                      │ │
│  │ $ sqlmap -u "http://site.com/page?id=1" --dbs              │ │
│  │                                                             │ │
│  │ Burp Suite (commercial):                                   │ │
│  │ • Intercept and modify requests                            │ │
│  │ • Automated scanning                                       │ │
│  │                                                             │ │
│  │ OWASP ZAP (open source):                                   │ │
│  │ • Proxy-based testing                                      │ │
│  │ • Active and passive scanning                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CODE REVIEW CHECKLIST                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ No string concatenation for SQL                         │ │
│  │ □ All queries use parameterized statements                │ │
│  │ □ ORM used correctly (no raw queries)                     │ │
│  │ □ Dynamic identifiers (columns) are allowlisted           │ │
│  │ □ Error messages don't expose database details            │ │
│  │ □ Database user has minimal privileges                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STATIC ANALYSIS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Semgrep rules for SQL injection                         │ │
│  │ • Bandit (Python)                                          │ │
│  │ • SonarQube                                                │ │
│  │ • CodeQL                                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
