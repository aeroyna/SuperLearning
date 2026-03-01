# Security Threats\n\n## Security Threats

### Overview

Malware, attacks, vulnerabilities

### Mechanisms

```c
typedef struct {
    int uid;
    int gid;
    int permissions;  // rwxrwxrwx
} AccessControl;

bool check_permission(AccessControl *acl, int uid, int operation) {
    // Check if user has permission for operation
    return (acl->permissions & operation) && (acl->uid == uid || acl->gid == user_gid);
}
```

### Implementation

**Linux Permissions**:
```bash
chmod 755 file.txt    # rwxr-xr-x
chown user:group file.txt
```

**Java Security**:
```java
import java.security.*;

SecurityManager sm = System.getSecurityManager();
if (sm != null) {
    sm.checkPermission(new FilePermission("/path", "read"));
}
```

**Python**:
```python
import os
import stat

# Check permissions
mode = os.stat('file.txt').st_mode
if mode & stat.S_IRUSR:
    print("Owner can read")
```

### Security Principles

- **Principle of least privilege**
- **Defense in depth**
- **Fail-safe defaults**
- **Complete mediation**

## Key Takeaways

1. Protection mechanisms enforce access control
2. Multiple layers of security are essential
3. Authentication verifies identity
4. Security is ongoing process, not one-time setup

## Interview Focus

1. Explain protection vs security
2. How do access control lists work?
3. What are common security threats?
4. Compare authentication methods
