# 10.3 Advanced Git Log

## Filtering by Content

```bash
# Find commits that added/removed a string
git log -S "function_name"

# Find commits that changed a regex pattern
git log -G "regex.*pattern"

# Show the actual changes
git log -S "function_name" -p
```

## Filtering by Author

```bash
# Commits by specific author
git log --author="John"

# Multiple authors
git log --author="John\|Jane"

# Exclude author
git log --author="^(?!John).*$"

# Commits by current user
git log --author="$(git config [user.name](http://user.name))"
```

## Filtering by Date

```bash
# Since date
git log --since="2024-01-01"
git log --since="2 weeks ago"
git log --since="yesterday"

# Until date
git log --until="2024-12-31"

# Date range
git log --since="2024-01-01" --until="2024-12-31"

# Last 30 days
git log --since="30 days ago"

# Specific time
git log --since="2024-01-15 10:30:00"
```

## Filtering by Message

```bash
# Commits mentioning "bug"
git log --grep="bug"

# Case insensitive
git log --grep="bug" -i

# Multiple patterns (OR)
git log --grep="bug" --grep="fix"

# Multiple patterns (AND)
git log --grep="bug" --grep="fix" --all-match

# Invert match (exclude)
git log --grep="WIP" --invert-grep
```

## Filtering by File/Path

```bash
# Commits that touched a file
git log file.txt

# Multiple files
git log file1.txt file2.txt

# Directory
git log src/

# Follow renames
git log --follow file.txt

# Show what changed in each commit
git log -p file.txt

# Show stats
git log --stat file.txt
```

## Filtering by Branch

```bash
# Commits in branch A but not in branch B
git log branchA ^branchB
# OR
git log branchB..branchA

# Commits in either branch A or B but not both
git log branchA...branchB

# Commits reachable from HEAD but not from main
git log main..HEAD

# Commits that will be pushed
git log origin/main..HEAD
```

## Finding Lost Commits

```bash
# Show all commits including unreachable ones
git log --all --reflog

# Find dangling commits
git fsck --lost-found

# Search reflog
git reflog | grep "commit message"
```

## Custom Formatting

```bash
# Custom format
git log --pretty=format:"%h - %an, %ar : %s"
# abc1234 - John Doe, 2 weeks ago : Fix bug

# Format options:
# %H  - Commit hash (full)
# %h  - Commit hash (short)
# %T  - Tree hash
# %P  - Parent hashes
# %an - Author name
# %ae - Author email
# %ad - Author date
# %ar - Author date (relative)
# %cn - Committer name
# %ce - Committer email
# %cd - Committer date
# %cr - Committer date (relative)
# %s  - Subject (commit message)
# %b  - Body

# Colors
git log --pretty=format:"%C(yellow)%h%C(reset) %s %C(blue)(%ar)%C(reset)"
```

## Finding Merge Commits

```bash
# Only merge commits
git log --merges

# Only non-merge commits
git log --no-merges

# First parent only (linear history)
git log --first-parent
```

## Finding Commits that Touch Function

```bash
# Track changes to a function
git log -L :function_name:file.js

# Shows every commit that modified that function
# With full diff of changes
```

## Statistical Analysis

```bash
# Commit count by author
git shortlog -sn

# Files changed most often
git log --pretty=format: --name-only | \
  sort | uniq -c | sort -rg | head -10

# Commits per day
git log --date=short --pretty=format:"%ad" | \
  sort | uniq -c

# Lines added/removed by author
git log --author="John" --pretty=tformat: --numstat | \
  awk '{add+=$1; subs+=$2} END {print "Added:",add,"Removed:",subs}'
```

## Complex Queries

```bash
# Commits by John that touched auth.js in last month
git log --author="John" \
        --since="1 month ago" \
        --grep="auth" \
        -- auth.js

# Bug fixes from last quarter
git log --grep="fix\|bug" -i \
        --since="3 months ago" \
        --pretty=format:"%h %s"

# Commits with large changes
git log --all --pretty=format:"%H %s" --shortstat | \
  awk '/files? changed/ {if ($4+$6 > 100) print prev} {prev=$0}'
```

## Finding When Bug Was Introduced

```bash
# Find when "bug_function" was added
git log -S "bug_function" --oneline

# See the actual change
git log -S "bug_function" -p

# Blame the current buggy line
git blame -L 42,42 buggy_file.js

# See that commit
git show abc1234

# Check what else changed
git show abc1234 --stat
```

## Performance Analysis

```bash
# Find commits that mention performance
git log -S "performance" --grep="slow\|perf\|optimize" -i

# Track file size changes
git log --all --pretty=format:"%H" -- large_file.bin | \
  xargs -I {} sh -c 'echo "{}: $(git show {}:large_file.bin | wc -c)"'
```

## Useful Log Aliases

```bash
# Add to ~/.gitconfig
[alias]
  lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
  ll = log --oneline --decorate --graph --all
  ls = log --pretty=format:"%C(yellow)%h %C(blue)%ad%C(red)%d %C(reset)%s%C(green) [%cn]" --decorate --date=short
  recent = log --oneline --decorate -10
```

## Advanced Log Techniques

### Find commits between tags

```bash
git log v1.0.0..v2.0.0 --oneline
```

### Commits unique to a branch

```bash
git log main..feature-branch --oneline
```

### Search commit content (not just messages)

```bash
git log -p -S "search term"
```

### Find commits that removed a file

```bash
git log --diff-filter=D --summary | grep delete
```

### Commits by hour of day

```bash
git log --pretty=format:"%ad" --date=format:"%H" | \
  sort | uniq -c | sort -n
```