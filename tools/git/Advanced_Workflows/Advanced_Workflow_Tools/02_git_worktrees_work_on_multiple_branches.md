# 7.2 Git Worktrees - Work on Multiple Branches

## What are Worktrees?

Work on multiple branches simultaneously without switching or stashing.

**Problem:** You're working on a feature, but need to fix a bug on main without losing your work.

**Solution:** Create a worktree!

## Basic Usage

```bash
# Main repo at /project
cd /project

# Create worktree for hotfix
git worktree add ../project-hotfix main

# Now you have:
# /project - your feature work
# /project-hotfix - clean main for hotfix

cd ../project-hotfix
git checkout -b hotfix-urgent
# Fix bug, commit, push

# Back to original
cd /project
# Your feature work is untouched!
```

## Creating Worktrees

```bash
# Create worktree for existing branch
git worktree add ../path branch-name

# Create worktree and new branch
git worktree add -b new-branch ../path

# Create from specific commit
git worktree add ../path abc1234
```

## Managing Worktrees

```bash
# List all worktrees
git worktree list

# Remove worktree
git worktree remove ../project-hotfix

# Prune stale worktrees
git worktree prune
```

## When to Use

✅ **Use for:**

- Reviewing PRs while working on feature
- Testing different branches simultaneously
- Hotfixes during feature work

❌ **Don't use for:**

- Simple branch switching
- When stash is sufficient