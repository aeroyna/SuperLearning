# 6.5 Reflog - Your Safety Net

## What is Reflog?

Reflog (reference log) tracks every movement of HEAD for 30-90 days. It's your safety net for recovering "lost" commits.

## Viewing Reflog

```bash
# See all HEAD movements
git reflog

# Output:
# abc1234 HEAD@{0}: reset: moving to HEAD~1
# def5678 HEAD@{1}: commit: My lost commit
# ghi9012 HEAD@{2}: commit: Another commit
# ...
```

Each entry shows:

- Commit hash
- HEAD position (HEAD@{n})
- Action that moved HEAD
- Commit message

## Recover from `reset --hard`

```bash
# You did this by mistake:
git reset --hard HEAD~3
# Oh no! Lost 3 commits!

# Don't panic! Check reflog
git reflog

# Find the commit before the reset
# abc1234 HEAD@{1}: commit: The commit I want back

# Recover it
git reset --hard abc1234
# OR create a branch at that commit:
git branch recovery abc1234
```

## Recover Deleted Branch

```bash
# Deleted a branch by mistake
git branch -D feature-branch
# Oh no!

# Find it in reflog
git reflog
# Look for the last commit on that branch

# Recreate the branch
git branch feature-branch abc1234
```

## Recover Dropped Stash

```bash
# Dropped a stash
git stash drop

# Find it in reflog
git reflog
# Look for: "WIP on branch: stash message"

# Recover it
git stash apply abc1234
```

## Filter Reflog

```bash
# Show reflog for specific branch
git reflog show main

# Show last 10 entries
git reflog -10

# Show reflog with dates
git reflog --date=iso
```

## Understanding Reflog Entries

```bash
git reflog

# abc1234 HEAD@{0}: commit: Latest commit
# def5678 HEAD@{1}: reset: moving to HEAD~1
# ghi9012 HEAD@{2}: commit (amend): Fixed commit
# jkl3456 HEAD@{3}: checkout: moving from main to feature
```

- `HEAD@{0}` = current position
- `HEAD@{1}` = one step back
- `HEAD@{2}` = two steps back

## Reflog Expiration

Reflog entries expire after:

- **90 days** for reachable commits
- **30 days** for unreachable commits

```bash
# Force garbage collection (removes expired reflog entries)
git reflog expire --expire=now --all
git gc --prune=now

# Don't do this unless you know what you're doing!
```

## Complete Recovery Example

```bash
# 1. Made commits
echo "important" > file.txt
git add file.txt
git commit -m "Important work"
echo "more important" >> file.txt
git commit -am "More important work"

# 2. Accidentally reset hard
git reset --hard HEAD~5
# All commits gone!

# 3. Check reflog
git reflog
# Find: def5678 HEAD@{1}: commit: More important work

# 4. Recover
git reset --hard def5678

# 5. Verify
git log
cat file.txt  # Your work is back!
```

<aside>
💾

**Reflog is your time machine!** As long as commits exist in reflog (30-90 days), you can recover them. This is why `reset --hard` isn't truly dangerous - reflog has your back.

</aside>

## Pro Tip: Create Recovery Branch

```bash
# Instead of resetting HEAD, create a branch
git reflog
git branch recovery abc1234

# Now you can examine the commit safely
git checkout recovery
git log

# If it's what you want, merge or reset
git checkout main
git reset --hard recovery
```