# 6.3 Using Git Reset

## Understanding `git reset` Modes

`git reset` has three modes that control what gets changed:

### 1. `--soft` (Safest)

```bash
git reset --soft HEAD~1
```

- **Moves branch pointer** back
- **Keeps staging area** (changes are staged)
- **Keeps working directory** (changes in files)

**Use case:** Redo the last commit with additional changes

### 2. `--mixed` (Default)

```bash
git reset HEAD~1
# Same as:
git reset --mixed HEAD~1
```

- **Moves branch pointer** back
- **Clears staging area** (changes unstaged)
- **Keeps working directory** (changes in files)

**Use case:** Undo commit and unstage, but keep changes to re-organize

### 3. `--hard` (Dangerous!)

```bash
git reset --hard HEAD~1
```

- **Moves branch pointer** back
- **Clears staging area**
- **Resets working directory** (changes GONE!)

**Use case:** Completely discard commits and changes

## Visual Representation

```
Initial state:
Commit:   A -- B -- C (HEAD, main)
Staging:  []
Working:  []

After git reset --soft B:
Commit:   A -- B (HEAD, main)
Staging:  [Changes from C]
Working:  [Changes from C]

After git reset --mixed B:
Commit:   A -- B (HEAD, main)
Staging:  []
Working:  [Changes from C]

After git reset --hard B:
Commit:   A -- B (HEAD, main)
Staging:  []
Working:  []  (Changes LOST!)
```

## Common Scenarios

### Undo Last Commit, Keep Changes Staged

```bash
# Made a commit
git commit -m "Add feature"

# Oops, forgot to add another file!
echo "more" > another.txt
git add another.txt

# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Now make a better commit
git add .
git commit -m "Add complete feature"
```

### Undo Last Commit, Keep Changes Unstaged

```bash
# Committed too early
git commit -m "WIP"

# Undo and unstage
git reset HEAD~1

# Changes are in working directory, ready to reorganize
git status
```

### Completely Remove Last Commit

```bash
# Made a terrible commit
git commit -m "Terrible mistake"

# Completely undo it (discard changes)
git reset --hard HEAD~1
```

<aside>
🚨

**Warning:** `--hard` discards ALL changes! Commit is gone, files are reverted. Only use when you're absolutely sure!

</aside>

## Undo Multiple Commits

```bash
# Go back 3 commits (keep changes staged)
git reset --soft HEAD~3

# All changes from those 3 commits are now staged
# Make one big commit:
git commit -m "Combine last 3 commits"
```

```bash
# Go back 3 commits (keep changes unstaged)
git reset HEAD~3

# Reorganize and commit properly
```

```bash
# Go back 3 commits (discard everything)
git reset --hard HEAD~3
# Last 3 commits GONE!
```

## Reset to Specific Commit

```bash
# Reset to a specific commit hash
git reset --soft abc1234
git reset --mixed def5678
git reset --hard ghi9012

# Reset to a tag
git reset --hard v1.0.0

# Reset to a branch
git reset --hard origin/main
```

## Safety Tip: Create Backup Branch

```bash
# Before risky reset, create backup
git branch backup

# Do your reset
git reset --hard HEAD~5

# If you mess up:
git reset --hard backup

# Delete backup when done
git branch -d backup
```