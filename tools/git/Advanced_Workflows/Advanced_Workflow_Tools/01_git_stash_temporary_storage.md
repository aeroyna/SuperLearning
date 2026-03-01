# 7.1 Git Stash - Temporary Storage

## What is Stashing?

Stash temporarily saves your uncommitted changes (both staged and unstaged) so you can switch contexts without committing.

**Scenario:** You're working on a feature, but need to quickly switch to `main` to fix a bug.

```bash
# You're on feature-branch with uncommitted changes
git status
# Modified: file1.txt, file2.txt

# Can't switch branches with uncommitted changes
git checkout main
# Error: Your local changes would be overwritten

# Solution: Stash them!
git stash
# Saved working directory and index state

# Now you can switch
git checkout main
# Fix the bug, commit, etc.

# Switch back and restore your work
git checkout feature-branch
git stash pop
# Your changes are back!
```

## Basic Stash Commands

```bash
# Stash changes
git stash
# OR with a message
git stash push -m "WIP: feature X"

# List all stashes
git stash list
# stash@{0}: WIP on feature: abc1234 commit message
# stash@{1}: WIP on main: def5678 other commit

# Apply most recent stash (keep it in stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply specific stash
git stash apply stash@{1}
git stash pop stash@{2}

# View stash contents
git stash show
git stash show -p  # Show full diff
git stash show stash@{1}

# Delete a stash
git stash drop stash@{0}

# Delete all stashes
git stash clear
```

## Advanced Stashing

```bash
# Stash only unstaged changes (keep staged)
git stash --keep-index

# Stash including untracked files
git stash -u
# OR
git stash --include-untracked

# Stash everything including ignored files
git stash -a
# OR
git stash --all

# Stash specific files
git stash push -m "Only important files" file1.txt file2.txt

# Create a branch from stash
git stash branch new-branch-name stash@{0}
# Creates new branch, applies stash, drops stash
```

## Stash Pop vs Apply

```bash
# pop = apply + drop
git stash pop
# If there are conflicts, stash is NOT dropped

# apply = keep stash for later
git stash apply
# Stash remains in list

# You can apply the same stash multiple times!
git stash apply stash@{0}  # On branch A
git checkout branch-B
git stash apply stash@{0}  # Same stash on branch B
```

## Handling Stash Conflicts

```bash
# Apply stash with conflicts
git stash pop
# CONFLICT in file.txt

# Resolve conflicts
vim file.txt
git add file.txt

# Stash is NOT automatically dropped after conflict
# Drop it manually:
git stash drop
```

## Practical Workflows

### Quick Context Switch

```bash
# Working on feature
git stash
git checkout main
git pull origin main
# Do urgent work
git checkout feature-branch
git stash pop
```

### Try Experimental Changes

```bash
# Make experimental changes
git stash
# Try different approach
# If better:
git stash drop
# If worse:
git stash pop  # Get original changes back
```

### Share Work Without Committing

```bash
# Create patch from stash
git stash show -p stash@{0} > my-changes.patch
# Send patch to teammate
# They apply it:
git apply my-changes.patch
```

## Stash Inspection

```bash
# List stashes with dates
git stash list --date=local

# Show files in stash
git stash show stash@{0} --name-only

# Show detailed diff
git stash show stash@{0} -p

# Show stash as patch
git stash show -p stash@{0} > stash.patch
```

## Common Stash Patterns

```bash
# Pattern 1: Keep working directory clean
git stash  # Before switching branches
git stash pop  # After returning

# Pattern 2: Test without committing
git stash  # Save current work
# Test something
git stash pop  # Restore work

# Pattern 3: Transfer changes between branches
git stash  # On branch A
git checkout branch-B
git stash pop  # Apply on branch B
```

<aside>
💡

**Pro tip:** Stash is perfect for when you need to switch contexts quickly but your work isn't ready to commit. Think of it as a clipboard for your uncommitted changes.

</aside>