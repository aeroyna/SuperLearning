# 6.4 Git Revert for Safe Undo

## When to Use Revert

Use `revert` when you need to undo a commit that's already been pushed to a shared branch. Unlike `reset`, `revert` doesn't rewrite history.

## Basic Revert

```bash
# Your history
git log --oneline
# c3c3c3 (HEAD) Good commit
# b2b2b2 BAD commit (want to undo this)
# a1a1a1 Good commit

# Create a NEW commit that undoes b2b2b2
git revert b2b2b2

# New history:
git log --oneline
# d4d4d4 (HEAD) Revert "BAD commit"
# c3c3c3 Good commit
# b2b2b2 BAD commit
# a1a1a1 Good commit
```

## Reset vs Revert

**Reset (rewrites history):**

```
Before: A -- B -- C -- D (main)
After:  A -- B (main)
        Commits C and D are gone!
```

**Revert (adds new commit):**

```
Before: A -- B -- C -- D (main)
After:  A -- B -- C -- D -- D' (main)
        D' undoes changes from D
        All history preserved!
```

<aside>
💡

**Key principle:**

- Use `reset` for local, unpushed commits
- Use `revert` for commits already pushed to shared branches
</aside>

## Revert Multiple Commits

```bash
# Revert a range (creates multiple revert commits)
git revert HEAD~3..HEAD

# This creates 3 separate revert commits
```

## Revert Without Committing

```bash
# Revert but don't commit immediately
git revert -n b2b2b2
# OR
git revert --no-commit b2b2b2

# Changes are staged, you can modify before committing
git status
git commit -m "Revert bad changes"
```

## Revert Multiple Commits as One

```bash
# Revert last 3 commits as a single revert commit
git revert -n HEAD~3..HEAD
git commit -m "Revert last 3 commits"
```

## Handling Revert Conflicts

```bash
# Try to revert
git revert abc1234

# If there are conflicts:
# CONFLICT: Merge conflict in file.txt

# Resolve conflicts
vim file.txt
git add file.txt

# Continue the revert
git revert --continue

# OR abort the revert
git revert --abort
```

## Revert a Merge Commit

Merge commits have two parents, so you need to specify which parent:

```bash
# Revert a merge commit
git revert -m 1 abc1234
# -m 1 means keep parent 1 (usually the main branch)
# -m 2 means keep parent 2 (usually the feature branch)
```

## When You Should Revert

✅ **Use revert when:**

- Commit is already pushed to shared branch
- Others have pulled your commit
- Working on `main` or `develop` branch
- You want to preserve history

❌ **Don't use revert when:**

- Commit is only local
- You're on a personal feature branch
- You want to clean up history before pushing