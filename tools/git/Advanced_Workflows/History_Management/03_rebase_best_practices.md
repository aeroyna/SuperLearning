# 8.3 Rebase Best Practices

## When to Rewrite History

✅ **Safe to rewrite:**

- Local commits not yet pushed
- Your own feature branches
- Before opening Pull Request
- Cleaning up experimental work

❌ **Never rewrite:**

- Commits on `main` or `develop`
- Commits already pushed to shared branches
- Commits others have based work on
- Public release tags

## The Golden Rules

<aside>
📜

**Rule 1:** Once you push, don't rewrite (unless it's your own feature branch)

**Rule 2:** Never rewrite public history (main, develop, release branches)

**Rule 3:** Communicate before force pushing to shared branches

</aside>

## Safe Rebase Workflow

```bash
# Local work - rebase freely
git checkout feature-branch
git rebase -i HEAD~5  # Clean up
git rebase main       # Update with main

# Now push
git push origin feature-branch

# After this point, avoid rebasing
# If you must rebase after pushing:
git push --force-with-lease origin feature-branch
```

## Force Push: The Right Way

### Never Use `--force`

```bash
# DANGEROUS - overwrites anything
git push --force

# Can destroy teammate's work!
```

### Always Use `--force-with-lease`

```bash
# SAFER - fails if someone else pushed
git push --force-with-lease

# Prevents overwriting teammate's work
```

**How it works:**

- Checks if remote branch matches your expectation
- Fails if someone else pushed changes
- Protects against accidental overwrites

## Communication is Key

If you must rewrite shared history:

1. **Announce to team** - "I need to rebase feature-X"
2. **Ensure nobody is working on it** - Check with teammates
3. **Use `--force-with-lease`** - Safety first
4. **Notify when done** - "Rebase complete, please pull"

## Rebase vs Merge Decision Tree

```
Are commits pushed and shared?
├─ Yes → Use MERGE (safe)
└─ No → Can use REBASE (local only)
    |
    Is branch public (main/develop)?
    ├─ Yes → Use MERGE (preserve history)
    └─ No → Use REBASE (clean history)
```

## Before You Rebase - Checklist

☑️ Commits are local only (not pushed)?

☑️ Or if pushed, it's your own feature branch?

☑️ Nobody else is working on this branch?

☑️ You've communicated with team?

☑️ You're prepared to force push?

If all checked, rebase is safe!

## Common Rebase Mistakes

### Mistake 1: Rebasing main

```bash
# WRONG - Never do this!
git checkout main
git rebase feature-branch

# This rewrites main's history!
```

**Instead:**

```bash
git checkout main
git merge feature-branch
```

### Mistake 2: Rebasing without coordination

```bash
# Team member is working on feature-branch
# You rebase and force push
git rebase main
git push --force origin feature-branch

# Their work is now based on old history!
```

**Instead:** Communicate first or use merge.

### Mistake 3: Using `--force` instead of `--force-with-lease`

```bash
# DANGEROUS
git push --force

# SAFER
git push --force-with-lease
```

## Recovery from Bad Rebase

```bash
# Oh no, I rebased main by mistake!

# Find the commit before rebase
git reflog
# abc1234 HEAD@{1}: checkout: moving from main to feature
# def5678 HEAD@{2}: rebase: Add feature
# ghi9012 HEAD@{3}: commit: Previous state

# Reset to before rebase
git reset --hard ghi9012

# Crisis averted!
```

## Team Workflow Recommendations

### Option 1: Rebase Before Merge (Clean History)

```bash
# Developer workflow:
git checkout feature-branch
git rebase main         # Update with main
git push --force-with-lease origin feature-branch

# Then merge on GitHub (creates merge commit)
# Result: Linear feature history + merge commit
```

### Option 2: Squash Merge (Simplest)

```bash
# Developer workflow:
git checkout feature-branch
# Make many commits (no cleanup needed)
git push origin feature-branch

# On GitHub: "Squash and merge"
# Result: One commit per feature in main
```

### Option 3: Merge Commits (Preserve Everything)

```bash
# Developer workflow:
git checkout feature-branch
# Make commits
git push origin feature-branch

# On GitHub: Regular merge
# Result: All commits preserved + merge commit
```

## When Rebase Goes Wrong

### Scenario: Conflicts During Rebase

```bash
# Start rebase
git rebase main
# CONFLICT in file.txt

# Option 1: Fix and continue
vim file.txt
git add file.txt
git rebase --continue

# Option 2: Skip this commit
git rebase --skip

# Option 3: Abort everything
git rebase --abort
```

### Scenario: Lost Commits After Rebase

```bash
# Commits seem gone after rebase
git reflog
# Find them and recover
git cherry-pick <lost-commit>
```

## Best Practices Summary

1. **Rebase early, rebase often** - Keep feature branch updated
2. **Clean up before pushing** - Interactive rebase locally
3. **Use `--force-with-lease`** - Never plain `--force`
4. **Communicate** - Tell team before rewriting shared history
5. **When in doubt, merge** - It's always safer
6. **Never rebase public branches** - main, develop, release
7. **Know reflog** - Your safety net for recovery

## Quick Reference

```bash
# Safe local rebase
git rebase main
git rebase -i HEAD~5

# Safe force push
git push --force-with-lease origin feature-branch

# Recovery
git reflog
git reset --hard HEAD@{2}

# Abort bad rebase
git rebase --abort
```