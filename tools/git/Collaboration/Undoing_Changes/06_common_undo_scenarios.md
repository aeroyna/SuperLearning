# 6.6 Common Undo Scenarios

## Restore File from Specific Commit

```bash
# Restore a file from 3 commits ago
git restore --source=HEAD~3 file.txt

# OR (old way):
git checkout HEAD~3 -- file.txt

# Restore from specific commit hash
git restore --source=abc1234 file.txt

# Restore from a tag
git restore --source=v1.0.0 file.txt

# Restore from another branch
git restore --source=feature-branch file.txt
```

## Discard All Local Changes

```bash
# Discard all changes and match remote exactly
git fetch origin
git reset --hard origin/main

# This discards:
# - All uncommitted changes
# - All local commits not on origin/main
```

## Undo `git add .` (Unstage Everything)

```bash
# Accidentally staged everything
git add .

# Unstage all
git restore --staged .
# OR
git reset
```

## Fix Wrong Commit Message

```bash
# Fix the last commit message (if not pushed)
git commit --amend -m "Correct message"

# If already pushed, you need to revert or force push (dangerous)
```

## Undo `git commit --amend`

```bash
# Amended by mistake
git reflog
# Find the commit before amend

# Reset to it
git reset --soft HEAD@{1}
```

## Remove Untracked Files

```bash
# See what would be removed
git clean -n

# Remove untracked files
git clean -f

# Remove untracked files and directories
git clean -fd

# Remove ignored files too
git clean -fdx
```

## Undo Merge (Before Commit)

```bash
# During merge with conflicts
git merge feature-branch
# CONFLICT!

# Abort the merge
git merge --abort
```

## Undo Merge (After Commit)

```bash
# Merged and committed
git merge feature-branch

# Undo the merge (not pushed yet)
git reset --hard HEAD~1

# OR if already pushed
git revert -m 1 HEAD
```

## Quick Decision Tree

**Discard changes in working directory?**

→ `git restore file.txt`

**Unstage file (keep changes)?**

→ `git restore --staged file.txt`

**Undo last commit, keep changes staged?**

→ `git reset --soft HEAD~1`

**Undo last commit, keep changes unstaged?**

→ `git reset HEAD~1`

**Completely remove last commit?**

→ `git reset --hard HEAD~1`

**Undo pushed commit?**

→ `git revert <commit>`

**Recover lost commits?**

→ `git reflog` then `git reset --hard <commit>`

**Get file from old commit?**

→ `git restore --source=<commit> file.txt`

**Match remote exactly?**

→ `git reset --hard origin/main`

## Safety Checklist

✅ **Before any destructive operation:**

1. Check status: `git status`
2. Check log: `git log --oneline`
3. Create backup branch: `git branch backup`
4. Remember reflog exists: `git reflog`

<aside>
⚠️

**Golden Rules:**

1. Never use `--hard` unless you're 100% sure
2. Never rewrite history on shared branches (use `revert`)
3. Always check `git status` before resetting
4. Reflog can save you (30-90 days)
5. When in doubt, create a backup branch first
</aside>

## Practice Exercises

```bash
# Setup
git init practice-undo
cd practice-undo
echo "v1" > file.txt
git add file.txt
git commit -m "V1"

# Exercise 1: Modify file, then restore
echo "v2" > file.txt
git restore file.txt
cat file.txt  # Should be "v1"

# Exercise 2: Stage and unstage
echo "v2" > file.txt
git add file.txt
git restore --staged file.txt
git status  # Should show "not staged"

# Exercise 3: Commit and undo with --soft
echo "v2" > file.txt
git commit -am "V2"
git reset --soft HEAD~1
git status  # Should show "staged"

# Exercise 4: Use reflog to recover
echo "v3" > file.txt
git commit -am "V3"
git reset --hard HEAD~1
cat file.txt  # Should be "v2"

# Now recover V3
git reflog
# Find V3 commit hash
git reset --hard <V3-hash>
cat file.txt  # Should be "v3" again!

# Exercise 5: Revert a commit
echo "bad" > file.txt
git commit -am "Bad commit"
git revert HEAD
cat file.txt  # Should be "v3" (undone)
```