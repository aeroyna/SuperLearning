# 10.2 Git Blame

## What is Git Blame?

Git blame shows who last modified each line of a file, when, and in which commit.

**Use cases:**

- Find who wrote a specific line of code
- Understand when a bug was introduced
- See the context of changes
- Find the commit message explaining why

## Basic Blame Usage

```bash
# Blame entire file
git blame file.txt

# Output:
# abc1234 (John Doe  2024-01-15 10:30:25 -0800  1) Line 1 content
# def5678 (Jane Smith 2024-02-20 14:22:10 -0800  2) Line 2 content
# abc1234 (John Doe  2024-01-15 10:30:25 -0800  3) Line 3 content
```

**Each line shows:**

- Commit hash
- Author name
- Date and time
- Line number
- Line content

## Blame Specific Lines

```bash
# Blame lines 10-20
git blame -L 10,20 file.txt

# Blame from line 10 to end
git blame -L 10 file.txt

# Blame lines around a function
git blame -L :function_name file.txt
```

## Simplified Blame Output

```bash
# Show only commit hash and line
git blame -s file.txt

# Show email instead of name
git blame -e file.txt

# Show relative dates
git blame --date=relative file.txt
# "2 weeks ago"

# Show short commit hashes
git blame --abbrev=8 file.txt
```

## Blame at Specific Commit

```bash
# See who modified lines at a specific point in history
git blame abc1234 file.txt

# Blame at a tag
git blame v1.0.0 file.txt

# Blame 5 commits ago
git blame HEAD~5 file.txt
```

## Ignoring Whitespace Changes

```bash
# Ignore whitespace changes
git blame -w file.txt

# Useful when someone reformatted the code
# Shows the "real" last change, not formatting
```

## Following Renames

```bash
# Track the file even if renamed
git blame -C file.txt

# More aggressive copy detection
git blame -C -C file.txt

# Even more aggressive (slower)
git blame -C -C -C file.txt
```

## Finding Who Deleted a Line

```bash
# Line was deleted, who did it?
# First, find when the line existed
git log -S "line content" file.txt

# Then blame that version
git blame abc1234 file.txt | grep "line content"
```

## Interactive Blame Investigation

```bash
# Who wrote line 42?
git blame -L 42,42 file.txt
# abc1234 (John Doe 2024-01-15) Line 42 content

# What was that commit about?
git show abc1234

# What else was changed in that commit?
git show abc1234 --stat

# See the file before that change
git show abc1234^:file.txt
```

## Blame in IDE/Editor

Most IDEs have git blame integrated:

**VS Code:**

- GitLens extension
- Hover over line to see blame info
- Click to see full commit

**IntelliJ/PyCharm:**

- Right-click line → Git → Annotate
- Shows blame in gutter

**Vim:**

```
:Git blame
```

## Real-World Example

```bash
# Bug in authentication code
# Find who wrote the problematic line

git blame src/auth.js | grep "validateToken"
# abc1234 (Alice 2024-03-10) validateToken(token, expired)

# Check the commit
git show abc1234
# Shows: "feat: add token expiry check"

# See the discussion
git log --all --grep="validateToken"

# Check if there's a related issue
git log --all --oneline | grep -i "token"
```

## Combining Blame with Log

```bash
# Find all commits that touched a specific function
git log -L :function_name:file.js

# Find who changed a specific line over time
git log -L 42,42:file.txt

# See evolution of a code section
git log -L 100,150:file.txt --oneline
```

## Blame Ignore File

Create `.git-blame-ignore-revs`:

```
# Ignore formatting commits
abc1234  # Black code formatting
def5678  # Prettier formatting
ghi9012  # Mass rename
```

Then:

```bash
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

Now blame skips those commits!

## Blame Best Practices

1. **Use blame to understand, not to criticize** - Focus on "why" not "who"
2. **Check commit message** - Often explains the reasoning
3. **Look at full commit** - Line might be part of larger change
4. **Ignore formatting commits** - Use `.git-blame-ignore-revs`
5. **Combine with log** - See full history of code section

## Common Blame Patterns

### Pattern 1: Find bug introduction

```bash
# Find buggy line
git blame file.js | grep "buggy_function"

# See commit
git show <commit-hash>

# Check tests at that time
git show <commit-hash>:tests/
```

### Pattern 2: Understand design decision

```bash
# Why was it done this way?
git blame file.js -L 100,120

# Read commit message
git show <commit-hash>

# See related commits
git log --all --grep="<keyword>"
```

### Pattern 3: Track code evolution

```bash
# See all changes to a function
git log -L :function_name:file.js -p

# Blame at different points
git blame HEAD file.js
git blame HEAD~10 file.js
git blame v1.0.0 file.js
```