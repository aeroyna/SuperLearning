# 7.3 Cherry-Pick - Selective Commits

## What is Cherry-Picking?

Apply a specific commit from one branch to another, without merging the entire branch.

```bash
# You're on main
git log --oneline feature-branch
# abc1234 Critical bug fix
# def5678 New feature
# ghi9012 Another change

# You only want the bug fix
git cherry-pick abc1234

# Now main has that commit
```

## Basic Cherry-Pick

```bash
# Cherry-pick single commit
git cherry-pick abc1234

# Cherry-pick multiple commits
git cherry-pick abc1234 def5678 ghi9012

# Cherry-pick range of commits
git cherry-pick abc1234..ghi9012
# Note: This does NOT include abc1234, only commits after it

# Cherry-pick range including start
git cherry-pick abc1234^..ghi9012
```

## Cherry-Pick with Options

```bash
# Cherry-pick without committing (stage changes)
git cherry-pick -n abc1234
# OR
git cherry-pick --no-commit abc1234

# Edit commit message during cherry-pick
git cherry-pick -e abc1234

# Add "cherry picked from" to commit message
git cherry-pick -x abc1234
# Useful for tracking where commit came from
# Adds: (cherry picked from commit abc1234)
```

## Handling Cherry-Pick Conflicts

```bash
# Cherry-pick with conflict
git cherry-pick abc1234
# CONFLICT in file.txt

# Resolve conflicts
vim file.txt
git add file.txt

# Continue cherry-pick
git cherry-pick --continue

# OR abort
git cherry-pick --abort

# OR skip this commit
git cherry-pick --skip
```

## Common Use Cases

### Hotfix to Multiple Branches

```bash
# Bug fixed on main
git checkout main
git commit -m "Fix critical bug"  # abc1234

# Apply to release branch
git checkout release-1.0
git cherry-pick abc1234

# Apply to release-2.0
git checkout release-2.0
git cherry-pick abc1234
```

### Extract Specific Feature

```bash
# Feature branch has 10 commits
# You only want commits 3, 5, 7

git checkout main
git cherry-pick commit3 commit5 commit7
```

### Undo Accidental Commit on Wrong Branch

```bash
# Committed on main by mistake
git log --oneline
# abc1234 (HEAD -> main) Should be on feature

# Create feature branch (preserves the commit)
git branch feature

# Remove from main
git reset --hard HEAD~1

# Commit is now only on feature
```

### Backport Fix to Old Release

```bash
# Fix in v2.0 needs to go to v1.5
git checkout v2.0
git log --oneline
# abc1234 Fix security issue

git checkout release-1.5
git cherry-pick abc1234
git push origin release-1.5
```

## Cherry-Pick Multiple Commits

```bash
# Cherry-pick range
git cherry-pick abc1234^..def5678

# Cherry-pick and edit each message
git cherry-pick -e abc1234 def5678 ghi9012

# Cherry-pick without committing (for manual combination)
git cherry-pick -n abc1234
git cherry-pick -n def5678
git commit -m "Combined fix"
```

## Cherry-Pick vs Merge vs Rebase

**Cherry-pick:** Select specific commits

```
Original:
main:     A -- B
feature:       \ C -- D -- E

Cherry-pick D:
main:     A -- B -- D'
```

**Merge:** Combine entire branches

```
main:     A -- B ---------- M
               \           /
feature:        C -- D -- E
```

**Rebase:** Move entire branch

```
main:     A -- B -- C' -- D' -- E'
```

## Tracking Cherry-Picked Commits

```bash
# Always use -x to track origin
git cherry-pick -x abc1234

# Commit message includes:
# (cherry picked from commit abc1234)

# Later, find all cherry-picked commits:
git log --grep="cherry picked from"
```

## Cherry-Pick from Another Repository

```bash
# Add other repo as remote
git remote add other-repo [https://github.com/user/other-repo.git](https://github.com/user/other-repo.git)
git fetch other-repo

# Cherry-pick from it
git cherry-pick other-repo/main~3
```

## Interactive Cherry-Pick

```bash
# Cherry-pick range but review each
git cherry-pick -n abc1234..def5678
# Review changes
git status
git diff --staged
# Commit when ready
git commit
```

<aside>
💡

**When to use cherry-pick:**

- Applying hotfixes to multiple release branches
- Extracting specific commits from a feature branch
- Moving commits between branches
- Backporting fixes to older versions

**When NOT to use:**

- When you need the entire branch (use merge)
- When you want linear history (use rebase)
- For regular integration workflows
</aside>

## Cherry-Pick Best Practices

1. **Use `-x` flag** to track where commits came from
2. **Test after cherry-picking** - context might be different
3. **Prefer merge for full branches** - cherry-pick for specific commits only
4. **Document why** you cherry-picked in commit messages
5. **Watch for dependencies** - cherry-picking one commit might break without others