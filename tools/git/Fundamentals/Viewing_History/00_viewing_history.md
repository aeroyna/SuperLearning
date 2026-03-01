# Chapter 3: Viewing History

In Perforce, you use `p4 changes` and `p4 filelog`. In git, history navigation is much more powerful.

## Basic Log Viewing

```bash
# View commit history (like p4 changes)
git log

# You'll see:
# commit abc123... (the commit hash - like a changelist number)
# Author: Your Name
# Date: ...
# Commit message
```

**Key difference:** Git uses SHA-1 hashes (40 characters) instead of sequential numbers. You only need the first 7-8 characters to reference a commit.

## Prettier Log Formats

```bash
# One line per commit (much cleaner!)
git log --oneline

# Shows:
# abc1234 Add readme file
# def5678 Initial commit

# With a visual graph (shows branching, we'll cover this later)
git log --oneline --graph --all

# My favorite format - detailed but compact
git log --oneline --decorate --graph --all
```

**Pro tip:** Create an alias for this!

```bash
git config --global alias.lg "log --oneline --graph --all --decorate"
# Now you can just type: git lg
```

## Filtering Logs

```bash
# Last 5 commits
git log -5

# Commits by author
git log --author="John"

# Commits in date range
git log --since="2 weeks ago"
git log --after="2024-01-01" --before="2024-12-31"

# Commits that modified a specific file (like p4 filelog)
git log readme.txt

# Search commit messages
git log --grep="bug fix"
```

## Viewing a Specific Commit

```bash
# Show details of a commit (like p4 describe)
git show abc1234

# Shows:
# - Commit metadata (author, date, message)
# - Full diff of what changed
```

```bash
# Show just the files that changed
git show --name-only abc1234

# Show stats (lines added/removed)
git show --stat abc1234
```

## Comparing Commits

```bash
# Diff between two commits
git diff abc1234 def5678

# Diff between a commit and current working directory
git diff abc1234

# Diff between commit and its parent (what that commit changed)
git diff abc1234^ abc1234
# OR shorter:
git show abc1234
```

## Who Changed What? (git blame)

```bash
# See who last modified each line (like p4 annotate)
git blame readme.txt

# Shows:
# abc1234 (John Doe 2024-01-15) line 1 content
# def5678 (Jane Smith 2024-02-20) line 2 content
```

```bash
# Blame for specific line range
git blame -L 10,20 readme.txt

# Blame at a specific commit (historical blame)
git blame abc1234 readme.txt
```

## Finding When Something Changed

```bash
# Find which commit introduced/removed a specific string
git log -S "function_name"

# Find commits that changed a specific line
git log -G "regex_pattern"
```

## Viewing File History

```bash
# See all commits that touched a file
git log --follow readme.txt
# --follow tracks the file even if it was renamed!

# See the actual changes to the file over time
git log -p readme.txt
# -p shows patches (diffs) for each commit
```

## Quick Reference Guide

**Perforce → Git:**

- `p4 changes` → `git log --oneline`
- `p4 describe 12345` → `git show abc1234`
- `p4 filelog file.txt` → `git log file.txt`
- `p4 annotate file.txt` → `git blame file.txt`

## Practical Scenarios

```bash
# "When did I last work on this feature?"
git log --since="1 month ago" --author="YourName"

# "What changed in the last release?"
git log v1.0..v2.0

# "Who broke this function?"
git blame -L 50,60 broken_file.js
git show <commit_hash>  # See full context

# "Show me everything that happened today"
git log --since="today" --all
```

## Understanding Commit References

```bash
# HEAD = current commit
git show HEAD

# HEAD^ or HEAD~ = parent of current commit
git show HEAD^

# HEAD~3 = 3 commits back
git show HEAD~3

# You can use these anywhere:
git diff HEAD~2 HEAD
git show HEAD^:readme.txt  # File content from parent commit
```

## Pro Tips

```bash
# See only merge commits
git log --merges

# See only non-merge commits
git log --no-merges

# See commits that aren't pushed yet
git log origin/main..HEAD

# Pretty format with custom info
git log --pretty=format:"%h - %an, %ar : %s"
# %h = short hash, %an = author name, %ar = relative date, %s = subject
```

## Try This Exercise

```bash
# Create some commits
echo "v1" > file.txt
git add file.txt
git commit -m "Version 1"

echo "v2" >> file.txt
git commit -am "Version 2"

echo "v3" >> file.txt
git commit -am "Version 3"

# Now explore:
git log --oneline
git show HEAD~1
git blame file.txt
git log -p file.txt
```