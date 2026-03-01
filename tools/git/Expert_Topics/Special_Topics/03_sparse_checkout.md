# 13.3 Sparse Checkout

## What is Sparse Checkout?

Sparse checkout allows you to checkout only specific directories from a large repo.

**Use case:** Monorepo with many projects, you only need one.

## Basic Sparse Checkout

```bash
# Clone without files
git clone --filter=blob:none --sparse [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)
cd monorepo

# Enable sparse checkout
git sparse-checkout init

# Set directories to checkout
git sparse-checkout set packages/frontend

# Now only packages/frontend is checked out
ls  # Only shows packages/frontend/
```

## Managing Sparse Checkout

```bash
# Add more directories
git sparse-checkout add packages/shared

# List current sparse checkout
git sparse-checkout list

# Disable sparse checkout (checkout everything)
git sparse-checkout disable

# Re-enable with previous settings
git sparse-checkout reapply
```

## Sparse Checkout Patterns

```bash
# Set multiple directories
git sparse-checkout set packages/frontend packages/shared

# Use wildcards
git sparse-checkout set "packages/*/src"

# Checkout everything except...
git sparse-checkout set "*" "!packages/legacy"
```

## Cone vs Non-Cone Mode

### Cone Mode (Default, Faster)

```bash
# Cone mode - matches whole directories
git sparse-checkout set packages/frontend

# More performant for large repos
# Simpler patterns
```

### Non-Cone Mode (More Flexible)

```bash
# Non-cone mode - more complex patterns
git sparse-checkout set --no-cone "packages/*/src" "!*.test.js"

# More flexible but slower
# Can match individual files
```

## Combining with Partial Clone

```bash
# Best for large repos
# Clone without blobs + sparse checkout
git clone --filter=blob:none --sparse [https://github.com/large/monorepo.git](https://github.com/large/monorepo.git)
cd monorepo
git sparse-checkout set packages/frontend

# Downloads only:
# - Commit metadata
# - Tree objects for sparse paths
# - Blobs for checked out files

# Very fast, minimal bandwidth!
```

## Sparse Checkout Use Cases

### Use Case 1: Monorepo - Single Project

```bash
# Work on just one project in monorepo
git clone --filter=blob:none --sparse [https://github.com/company/monorepo.git](https://github.com/company/monorepo.git)
cd monorepo
git sparse-checkout set packages/my-project
```

### Use Case 2: Docs Only

```bash
# Just need documentation
git clone --filter=blob:none --sparse [https://github.com/project/repo.git](https://github.com/project/repo.git)
cd repo
git sparse-checkout set docs/
```

### Use Case 3: CI/CD - Specific Service

```bash
# CI only needs to build one service
git clone --filter=blob:none --sparse [https://github.com/company/services.git](https://github.com/company/services.git)
cd services
git sparse-checkout set services/auth
cd services/auth
# Build and deploy...
```

## Sparse Checkout Best Practices

1. **Combine with partial clone** - Use --filter=blob:none
2. **Use cone mode** - Faster for most cases
3. **Include necessary files** - README, config files
4. **Document patterns** - Keep list of sparse patterns
5. **Test CI/CD** - Ensure builds work with sparse checkout
6. **Update as needed** - Add paths when needed

## Troubleshooting

```bash
# Files not appearing?
git sparse-checkout list  # Check patterns

# Restore full checkout
git sparse-checkout disable

# Reset sparse checkout
git sparse-checkout init --cone
git sparse-checkout set packages/frontend
```