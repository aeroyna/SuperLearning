# 4.4 Merge Conflicts

## What Are Merge Conflicts?

Conflicts happen when both branches modify the **same lines** in the **same file** after diverging.

## Creating a Conflict Scenario

<aside>
⚠️

**Key requirement for conflicts:** Both branches must modify the same file **after** they've diverged from a common ancestor.

</aside>

### Step-by-Step Conflict Creation

```bash
# Start on main, create a file
git checkout main
echo "original content" > conflict.txt
git add conflict.txt
git commit -m "Add conflict.txt"

# Create a branch (feature3 now points to the same commit as main)
git checkout -b feature3

# Go BACK to main and modify the file
git checkout main
echo "version from main" > conflict.txt
git add conflict.txt
git commit -m "Main modifies conflict.txt"
# main moved forward

# Go to feature3 and modify the SAME file differently
git checkout feature3
echo "version from feature" > conflict.txt
git add conflict.txt
git commit -m "Feature modifies conflict.txt"
# feature3 moved forward

# NOW we have divergence:
#     main (modified conflict.txt)
#      |
#      v
# A -- B  
#  \
#   C  (feature3 modified conflict.txt differently)
#    ^
#  feature3

# Try to merge - CONFLICT!
git checkout main
git merge feature3
```

## What You'll See

```
Auto-merging conflict.txt
CONFLICT (content): Merge conflict in conflict.txt
Automatic merge failed; fix conflicts and then commit the result.
```

## Understanding the Conflict Markers

```bash
cat conflict.txt
```

You'll see:

```
<<<<<<< HEAD
version from main
=======
version from feature
>>>>>>> feature3
```

Git marked the conflicting sections:

- `<<<<<<< HEAD` - Your current branch (main)
- `=======` - The divider
- `>>>>>>> feature3` - The branch you're merging

## Resolving Conflicts

### Option 1: Manually Edit the File

Remove the markers and keep what you want:

```bash
# Edit conflict.txt to your desired final version
echo "final resolved version" > conflict.txt

# Stage the resolved file
git add conflict.txt

# Commit the merge
git commit -m "Resolve conflict"
# Git already prepared a merge commit message for you
```

### Option 2: Choose One Side Completely

```bash
# Keep main's version
git checkout --ours conflict.txt

# OR keep feature3's version
git checkout --theirs conflict.txt

# Then stage and commit
git add conflict.txt
git commit
```

## Checking Status During Conflict

```bash
git status
# Shows:
# - Which files have conflicts ("both modified")
# - What's been resolved
# - Instructions on what to do next
```

## Aborting a Merge

If you want to bail out:

```bash
git merge --abort
# Goes back to before you started merging
```

## Complete Conflict Resolution Example

```bash
# Create conflict scenario
git checkout main
echo "line 1" > file.txt
git add file.txt
git commit -m "Add file"

git checkout -b feature
git checkout main
echo "line 1 - main version" > file.txt
git commit -am "Main changes"

git checkout feature
echo "line 1 - feature version" > file.txt
git commit -am "Feature changes"

# Merge and get conflict
git checkout main
git merge feature
# CONFLICT!

# Check the file
cat file.txt

# Resolve it
echo "line 1 - final version" > file.txt
git add file.txt
git commit -m "Resolve merge conflict"

# Check the result
git log --oneline --graph --all
```