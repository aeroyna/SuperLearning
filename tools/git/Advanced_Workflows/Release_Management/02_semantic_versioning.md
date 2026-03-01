# 9.2 Semantic Versioning

## What is Semantic Versioning?

Semantic Versioning (SemVer) is a versioning scheme that gives meaning to version numbers.

**Format:** `MAJOR.MINOR.PATCH` (e.g., `v2.4.1`)

```
v2.4.1
│ │ │
│ │ └─ PATCH: Bug fixes, backward compatible
│ └─── MINOR: New features, backward compatible
└───── MAJOR: Breaking changes, not backward compatible
```

## Version Number Rules

### MAJOR version (v2.0.0)

**When to bump:**

- Breaking API changes
- Removes features
- Changes behavior in incompatible ways
- User code will break without updates

**Examples:**

- Removing a public function
- Changing function parameters
- Renaming configuration options
- Dropping support for old versions

### MINOR version (v1.5.0)

**When to bump:**

- New features added
- New APIs introduced
- Functionality enhanced
- Backward compatible (old code still works)

**Examples:**

- Adding new functions
- Adding optional parameters
- New configuration options
- Performance improvements (non-breaking)

### PATCH version (v1.4.3)

**When to bump:**

- Bug fixes only
- Security patches
- Documentation fixes
- Backward compatible

**Examples:**

- Fixing crash bugs
- Correcting calculations
- Fixing memory leaks
- Security vulnerabilities

## Starting Versions

```bash
# Initial development
0.1.0  # First working version
0.2.0  # More features
0.9.0  # Almost ready

# First public release
1.0.0  # Stable API

# Bug fix
1.0.1

# New feature
1.1.0

# Breaking change
2.0.0
```

<aside>
📌

**Rule:** Version 0.x.y is for initial development. Anything can change. API is not stable.

</aside>

## Pre-release Versions

```bash
# Alpha (internal testing)
1.0.0-alpha
1.0.0-alpha.1

# Beta (external testing)
1.0.0-beta
1.0.0-beta.2

# Release Candidate
1.0.0-rc.1
1.0.0-rc.2

# Final release
1.0.0
```

**Order:** `1.0.0-alpha < 1.0.0-beta < 1.0.0-rc < 1.0.0`

## Build Metadata

```bash
# Add build information (doesn't affect precedence)
1.0.0+20241205
1.0.0+build.123
1.0.0-beta+exp.sha.5114f85
```

## Real-World Examples

### Example 1: Web API

```bash
v1.0.0  # Initial release: GET /users, POST /users
v1.1.0  # Added: GET /users/:id
v1.1.1  # Fixed: User creation bug
v1.2.0  # Added: PUT /users/:id
v2.0.0  # Breaking: Changed /users response format
```

### Example 2: Library

```bash
v1.0.0  # function calculate(a, b)
v1.1.0  # function calculate(a, b, options = {})  # Backward compatible
v1.1.1  # Fixed: calculation bug
v2.0.0  # function calculate(config)  # Breaking: different signature
```

## Deciding Version Bumps

**Decision tree:**

1. **Did I break backward compatibility?**
    - Yes → Bump MAJOR (2.0.0)
    - No → Continue
2. **Did I add new features?**
    - Yes → Bump MINOR (1.5.0)
    - No → Continue
3. **Did I fix bugs only?**
    - Yes → Bump PATCH (1.4.3)

## Version Ranges (Dependencies)

```bash
# Exact version
1.2.3

# Greater than
>1.2.3

# Greater than or equal
>=1.2.3

# Less than
<2.0.0

# Tilde (patch updates allowed)
~1.2.3  # Means >=1.2.3 <1.3.0

# Caret (minor updates allowed)
^1.2.3  # Means >=1.2.3 <2.0.0

# Range
>=1.2.3 <2.0.0
```

## Version in Code

### package.json (Node.js)

```json
{
  "name": "my-app",
  "version": "1.4.2",
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

### [setup.py](http://setup.py) (Python)

```python
setup(
    name='my-package',
    version='2.1.0',
)
```

### pom.xml (Java/Maven)

```xml
<version>1.3.5</version>
```

## Automated Versioning

```bash
# Using npm
npm version patch  # 1.0.0 -> 1.0.1
npm version minor  # 1.0.1 -> 1.1.0
npm version major  # 1.1.0 -> 2.0.0

# Automatically creates git tag
```

## Conventional Commits (Automation)

Format: `<type>(<scope>): <description>`

```bash
# Patch version (bug fix)
git commit -m "fix: resolve login issue"
git commit -m "fix(auth): correct token validation"

# Minor version (new feature)
git commit -m "feat: add user export"
git commit -m "feat(api): add GET /users endpoint"

# Major version (breaking change)
git commit -m "feat!: remove deprecated API"
git commit -m "BREAKING CHANGE: remove support for v1 API"
```

Tools like `semantic-release` can read these and auto-version.

## SemVer Best Practices

1. **Start with 0.1.0** for initial development
2. **Release 1.0.0** when API is stable
3. **Document breaking changes** clearly
4. **Never break PATCH versions** - only bug fixes
5. **Use pre-releases** for testing (alpha, beta, rc)
6. **Keep CHANGELOG** updated
7. **Follow consistently** - don't skip versions