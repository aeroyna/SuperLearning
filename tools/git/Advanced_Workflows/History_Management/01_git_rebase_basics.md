# 8.1 Git Rebase Basics

## What is Rebasing?

Rebasing moves or combines commits to create a linear history.

## Merge vs Rebase

**Merge (creates merge commit):**

```
main:     A -- B -- D -- M
               \       /
feature:        C1 -- C2
```

**Rebase (linear history):**

```
main:     A -- B -- D -- C1' -- C2'
```

## Basic Rebase

```bash
# You're on feature-branch
git checkout feature-branch

# Rebase onto main
git rebase main

# What happens:
# 1. Git finds common ancestor of feature and main
# 2. Saves your commits (C1, C2) temporarily
# 3. Resets feature to match main
# 4. Replays your commits on top of main
```

**Visual:**

```
Before rebase:
main:     A -- B -- D
               \
feature:        C1 -- C2

After rebase:
main:     A -- B -- D
                      \
feature:               C1' -- C2'
```

## When to Rebase

✅ **Use rebase when:**

- Updating your feature branch with latest main
- Cleaning up local commits before pushing
- Creating linear history
- Working on personal branches

❌ **Never rebase when:**

- Branch is already pushed and shared
- Others are working on the same branch
- On public branches like `main` or `develop`

<aside>
🚨

**Golden rule:** Never rebase commits that exist outside your repository!

</aside>

## Rebase Workflow

```bash
# Update feature branch with latest main
git checkout feature-branch
git fetch origin
git rebase origin/main

# If conflicts occur:
# Fix conflicts in files
vim file.txt
git add file.txt
git rebase --continue

# OR skip this commit
git rebase --skip

# OR abort rebase
git rebase --abort
```

## Handling Rebase Conflicts

```bash
# During rebase, conflict occurs
# CONFLICT (content): Merge conflict in file.txt

# 1. Check which files have conflicts
git status

# 2. Fix conflicts
vim file.txt
# Remove conflict markers

# 3. Stage resolved files
git add file.txt

# 4. Continue rebase
git rebase --continue

# Repeat for each conflicting commit
```

## Rebase onto Different Base

```bash
# Move feature from old-main to new-main
git rebase --onto new-main old-main feature-branch

# Visual:
Before:
old-main: A -- B -- C
                \
feature:         D -- E

new-main: A -- B -- C -- F -- G

After:
new-main: A -- B -- C -- F -- G
                              \
feature:                       D' -- E'
```

## Force Push After Rebase

```bash
# After rebasing pushed commits (be careful!)
git push --force-with-lease origin feature-branch

# --force-with-lease is safer than --force
# It fails if someone else pushed to the branch
```

<aside>
⚠️

**Warning:** Force pushing rewrites remote history. Only do this on your own feature branches, never on shared branches like main!

</aside>

## Rebase vs Merge - When to Use What

**Use Merge when:**

- Integrating feature into main
- Want to preserve branch history
- Working with public/shared branches
- Want to see when features were integrated

**Use Rebase when:**

- Updating feature with latest main
- Cleaning up local commits
- Want linear history
- Before opening Pull Request

**Typical workflow:**

```bash
# While developing feature:
git rebase origin/main  # Keep up to date

# When done:
# Open Pull Request on GitHub
# Merge via GitHub (merge commit or squash)
```

## Common Rebase Scenarios

### Scenario 1: Update feature with main

```bash
# Main has new commits you need
git checkout feature-branch
git rebase main

# Your commits now sit on top of latest main
```

### Scenario 2: Rebase before PR

```bash
# Clean up before opening PR
git checkout feature-branch
git rebase -i HEAD~5  # Interactive rebase
# Squash/clean up commits
git push origin feature-branch
```

### Scenario 3: Sync with updated main

```bash
# Main changed while you worked
git fetch origin
git rebase origin/main
# Fix any conflicts
git push --force-with-lease origin feature-branch
```