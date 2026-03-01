# 13.5 Advanced Techniques

## Partial Clone Strategies

### Blobless Clone

```bash
# Clone without blobs (files downloaded on demand)
git clone --filter=blob:none [https://github.com/large/repo.git](https://github.com/large/repo.git)

# Pros: Much faster clone, smaller initial size
# Cons: Files downloaded when checked out
```

### Treeless Clone

```bash
# Clone without any tree objects
git clone --filter=tree:0 [https://github.com/large/repo.git](https://github.com/large/repo.git)

# Most extreme: Only commits and tags
# Trees downloaded on demand
```

### Shallow Clone

```bash
# Clone with limited history
git clone --depth=1 [https://github.com/large/repo.git](https://github.com/large/repo.git)

# Only latest commit
# Much faster, much smaller

# Unshallow later if needed
git fetch --unshallow
```

### Combining Strategies

```bash
# Shallow + blobless
git clone --depth=1 --filter=blob:none [https://github.com/repo.git](https://github.com/repo.git)

# Shallow + sparse
git clone --depth=1 --sparse [https://github.com/repo.git](https://github.com/repo.git)
cd repo
git sparse-checkout set src/
```

## Git Bundles - Offline Transfer

```bash
# Create bundle (entire repo)
git bundle create repo.bundle --all

# Create bundle (specific branch)
git bundle create main.bundle main

# Create bundle (tag range)
git bundle create updates.bundle v1.0.0..v2.0.0

# Clone from bundle
git clone repo.bundle repo-copy

# Fetch from bundle
git fetch repo.bundle

# Verify bundle integrity
git bundle verify repo.bundle

# Incremental bundle (only new commits)
git bundle create update.bundle main ^origin/main
```

## Git Notes - Attach Metadata

```bash
# Add note to commit
git notes add -m "Reviewed by: John" abc1234

# View notes in log
git log --show-notes

# Edit note
git notes edit abc1234

# Show specific note
git notes show abc1234

# Remove note
git notes remove abc1234

# List all notes
git notes list

# Push notes to remote
git push origin refs/notes/*

# Fetch notes from remote
git fetch origin refs/notes/*:refs/notes/*
```

### Notes Use Cases

```bash
# Code review metadata
git notes add -m "Reviewed-by: Alice <[alice@example.com](mailto:alice@example.com)>" HEAD

# CI/CD results
git notes add -m "Build: PASSED, Tests: 150/150" HEAD

# Deployment tracking
git notes add -m "Deployed to production: 2024-12-05" abc1234
```

## Git Replace - Substitute Commits

```bash
# Replace one commit with another
git replace abc1234 def5678

# Now abc1234 behaves like def5678
# Useful for grafting history

# List replacements
git replace -l

# Remove replacement
git replace -d abc1234

# Push replacements
git push origin refs/replace/*
```

## Maintenance and Optimization

```bash
# Run garbage collection
git gc

# Aggressive garbage collection
git gc --aggressive --prune=now

# Optimize repository
git repack -Ad
git prune

# Verify repository integrity
git fsck --full

# Show repo size
git count-objects -vH

# Clean up unreachable objects
git reflog expire --expire=now --all
git gc --prune=now
```

## Repository Statistics

```bash
# Count objects in repo
git count-objects -vH

# Find largest files in history
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sed -n 's/^blob //p' | \
  sort --numeric-sort --key=2 | \
  tail -n 10

# Count commits by author
git shortlog -sn --all --no-merges

# Files changed most often
git log --pretty=format: --name-only | \
  sort | uniq -c | sort -rg | head -10
```

## Advanced Diff Techniques

```bash
# Word diff (better for prose)
git diff --word-diff

# Character diff
git diff --color-words

# Ignore whitespace
git diff -w

# Show function context
git diff -W

# Compare with patience algorithm
git diff --patience
```

## Advanced Worktree Techniques

```bash
# Permanent worktrees for environments
git worktree add /deployments/staging staging
git worktree add /deployments/production main

# Each environment has its own directory
cd /deployments/production
git pull
npm run build
# Deploy...
```