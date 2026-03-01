# Chapter 2: Basic Workflow

## The Git Workflow

The fundamental pattern is: **Edit → Stage → Commit**

## Creating and Tracking Files

```bash
# Create a file
echo "Hello Git" > readme.txt

# Check status
git status
# You'll see: "Untracked files: readme.txt"
```

In Perforce, new files need `p4 add`. Git sees them but doesn't track until you tell it to.

## Staging (the Index)

This is git's unique feature - you choose EXACTLY what goes in each commit:

```bash
# Stage the file (add to staging area)
git add readme.txt

# Check status again
git status
# Now: "Changes to be committed: new file: readme.txt"
```

**Why staging exists:** You can make 10 changes, but commit them in 3 logical chunks. Perforce changelists are similar, but git's staging is more flexible.

## Committing

```bash
# Commit (like p4 submit, but LOCAL only)
git commit -m "Add readme file"

# Check status
git status
# "nothing to commit, working tree clean"
```

**Key difference:** This commit is ONLY on your machine. No server involved yet.

## Making More Changes

```bash
# Edit the file
echo "Learning git is fun" >> readme.txt

# Also create another file
echo "print('hello')" > [script.py](http://script.py)

# Check what changed
git status
# Shows: modified readme.txt, untracked [script.py](http://script.py)
```

## Viewing Differences

```bash
# See what changed in tracked files (like p4 diff)
git diff readme.txt

# Stage the changes
git add readme.txt

# Now diff won't show anything (it's staged!)
# To see staged changes:
git diff --staged
```

## Selective Staging

```bash
# Stage just one file
git add readme.txt

# Commit it
git commit -m "Update readme"

# [script.py](http://script.py) is still untracked - you can commit it separately later
```

## Shortcuts

```bash
# Stage ALL modified files at once
git add .

# Stage and commit in one go (only for tracked files)
git commit -am "Quick commit of all changes"
```

<aside>
⚠️

**Common gotcha:** `git commit` without `-a` only commits STAGED changes. If you modify a file after staging, you need to `git add` again!

</aside>

## Quick Practice

```bash
echo "line 1" > test.txt
git add test.txt
git commit -m "Add test"

echo "line 2" >> test.txt
git status  # Modified but not staged
git add test.txt
git commit -m "Add line 2"
```

## Perforce Translation Guide

- `p4 edit` → not needed, just edit
- `p4 add` → `git add`
- `p4 submit` → `git commit` (local) + `git push` (to server, covered later)
- `p4 diff` → `git diff`

## Understanding Tracked vs Untracked

```bash
# NEW file → untracked
echo "new" > file.txt
git status  # "Untracked files: file.txt"

# Stage it once
git add file.txt
git commit -m "Add file"
# Now file.txt is TRACKED forever (git watches it)

# Edit the tracked file
echo "change" >> file.txt
git status  # "Changes not staged: modified: file.txt"
# It's tracked, but the changes aren't staged yet
```

## Key Insights

- **Tracked** = git knows about the file (it's in the repository history)
- **Staged** = changes are ready for the next commit (sitting in the index)

## The Staging Mental Model

Think of staging as a **snapshot preview**:

```bash
echo "A" > file.txt
git add file.txt          # Staged: "A"

echo "B" >> file.txt      # Working dir now has "A\nB"
git status                # Shows BOTH staged and unstaged changes!

# If you commit now, only "A" gets committed
# The "B" is still in your working directory but not staged
```

Every time you `git add`, you're saying "take a snapshot of THIS version for the next commit."

## Practical Pattern

```bash
# Make multiple edits
vim file1.txt
vim file2.txt
vim file3.txt

# Stage them incrementally
git add file1.txt
git add file2.txt
# Decide file3 isn't ready yet

git commit -m "Update files 1 and 2"
# file3.txt changes stay in working directory, uncommitted
```

This is **way more flexible** than Perforce where everything in your changelist submits together.