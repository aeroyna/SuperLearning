# 11.3 Monorepo Strategies

## What is a Monorepo?

A monorepo is a single repository containing multiple projects/packages.

**Examples:**

- Google: Entire codebase in one repo
- Facebook: React, React Native, Jest, etc.
- Babel: All packages in one repo

**Structure:**

```
monorepo/
├── packages/
│   ├── frontend/
│   ├── backend/
│   ├── shared/
│   └── mobile/
├── tools/
├── docs/
└── package.json
```

## Monorepo vs Polyrepo

### Monorepo Benefits

✅ **Advantages:**

- Atomic changes across projects
- Easier code sharing
- Single source of truth
- Simplified dependency management
- Better refactoring across projects
- Consistent tooling
- Single CI/CD pipeline

### Monorepo Challenges

❌ **Challenges:**

- Large repo size
- Slower clone/operations
- More complex CI/CD
- Harder access control
- Need specialized tools
- Can be overwhelming for new developers

## Sparse Checkout - Only Get What You Need

```bash
# Clone without files
git clone --filter=blob:none --sparse [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)
cd monorepo

# Only checkout specific directory
git sparse-checkout set packages/frontend

# Now only packages/frontend is checked out

# Add more directories
git sparse-checkout add packages/shared

# List current sparse checkout
git sparse-checkout list
```

## Partial Clone

```bash
# Clone without blobs (download on demand)
git clone --filter=blob:none [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)

# Clone without history (shallow)
git clone --depth=1 [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)

# Combine both
git clone --filter=blob:none --depth=1 [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)

# Clone with sparse checkout
git clone --filter=blob:none --sparse [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)
```

## Working in Monorepo

```bash
# See what changed in specific directory
git log packages/frontend/

# Blame specific package
git blame packages/backend/src/auth.js

# Create branch for package-specific work
git checkout -b feature/frontend-redesign

# Filter commits by path
git log --oneline -- packages/frontend/

# Show stats for specific package
git log --stat -- packages/frontend/
```

## Monorepo Tools

### Lerna (JavaScript/TypeScript)

```bash
# Install lerna
npm install -g lerna

# Initialize
lerna init

# Run command in all packages
lerna run build
lerna run test

# Version and publish
lerna version
lerna publish
```

### Nx (JavaScript/TypeScript)

```bash
# More advanced than Lerna
# Intelligent build system

# Only build what changed
nx affected:build

# Only test what changed
nx affected:test

# Show dependency graph
nx dep-graph
```

### Turborepo (JavaScript/TypeScript)

```bash
# Fast build system
# Remote caching

# Build all packages
turbo run build

# Run with caching
turbo run build --cache-dir=.turbo
```

### Bazel (Any language)

- Google's build tool
- Language agnostic
- Incremental builds
- Remote caching
- Very powerful but complex

## Monorepo CI/CD

```yaml
# GitHub Actions - only run tests for changed packages
name: CI
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Full history for comparison
      
      - name: Get changed files
        id: changed-files
        run: |
          echo "files=$(git diff --name-only HEAD~1)" >> $GITHUB_OUTPUT
      
      - name: Test frontend
        if: contains(steps.changed-files.outputs.files, 'packages/frontend')
        run: cd packages/frontend && npm test
      
      - name: Test backend
        if: contains(steps.changed-files.outputs.files, 'packages/backend')
        run: cd packages/backend && npm test
```

## Managing Dependencies in Monorepo

### Workspaces (npm/yarn/pnpm)

```json
// Root package.json
{
  "name": "monorepo",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "devDependencies": {
    "typescript": "^5.0.0",
    "jest": "^29.0.0"
  }
}
```

### Package-specific dependencies

```json
// packages/frontend/package.json
{
  "name": "@company/frontend",
  "version": "1.0.0",
  "dependencies": {
    "@company/shared": "*",
    "react": "^18.0.0"
  }
}
```

### Installing dependencies

```bash
# Install all workspace dependencies
npm install

# Add dependency to specific package
npm install axios --workspace=packages/frontend

# Run script in specific workspace
npm run build --workspace=packages/frontend
```

## Code Ownership in Monorepo

```bash
# CODEOWNERS file
packages/frontend/       @frontend-team
packages/backend/        @backend-team
packages/shared/         @platform-team
packages/mobile/         @mobile-team
```

## Monorepo Best Practices

1. **Use sparse checkout** for large repos
2. **Implement affected builds** - only build what changed
3. **Shared tooling** at root level
4. **Clear ownership** - CODEOWNERS file
5. **Path-based access control** if needed
6. **Good CI/CD** - parallel, incremental builds
7. **Documentation** - clear structure and conventions
8. **Consistent naming** - @company/package-name
9. **Shared configs** - ESLint, TypeScript, etc.
10. **Regular cleanup** - remove unused code

## Monorepo Structure Patterns

### Pattern 1: By Application

```
monorepo/
├── apps/
│   ├── web/
│   ├── mobile/
│   └── admin/
├── packages/
│   ├── ui-components/
│   ├── utils/
│   └── api-client/
└── tools/
```

### Pattern 2: By Layer

```
monorepo/
├── frontend/
├── backend/
├── shared/
├── infrastructure/
└── docs/
```

### Pattern 3: By Domain

```
monorepo/
├── auth/
│   ├── frontend/
│   ├── backend/
│   └── shared/
├── payments/
│   ├── frontend/
│   ├── backend/
│   └── shared/
└── shared/
    └── ui/
```

## Monorepo Scripts

```json
// Root package.json
{
  "scripts": {
    "build": "lerna run build",
    "test": "lerna run test",
    "lint": "lerna run lint",
    "clean": "lerna clean",
    "dev:frontend": "npm run dev --workspace=packages/frontend",
    "dev:backend": "npm run dev --workspace=packages/backend"
  }
}
```

## When to Use Monorepo

✅ **Use monorepo when:**

- Multiple projects share code
- Need atomic changes across projects
- Small to medium team (< 100 developers)
- Good tooling available
- Team can handle complexity

❌ **Use polyrepo when:**

- Projects are completely independent
- Different teams with different processes
- Very large organization
- Strict access control needed
- Simpler workflow preferred