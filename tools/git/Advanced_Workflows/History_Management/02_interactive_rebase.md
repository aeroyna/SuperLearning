# 8.2 Interactive Rebase

## What is Interactive Rebase?

Interactive rebase lets you edit, reorder, squash, and clean up commits before sharing them.

## Starting Interactive Rebase

```bash
# Rebase last 4 commits
git rebase -i HEAD~4

# OR rebase from a specific commit
git rebase -i abc1234

# OR rebase everything since branching from main
git rebase -i main
```

## The Interactive Editor

When you run `git rebase -i`, an editor opens:

```
pick abc1234 Add login feature
pick def5678 Fix typo
pick ghi9012 Fix another typo
pick jkl3456 Add tests

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous
# f, fixup = like squash, but discard message
# d, drop = remove commit
# x, exec = run command using shell
```

## Command: pick (default)

Keep the commit as-is.

```bash
pick abc1234 Add login feature
pick def5678 Add tests
```

Nothing changes - commits stay the same.

## Command: reword - Change Message

```bash
pick abc1234 Add login feature
reword def5678 Add tests  # Change this message
```

Git will stop and let you edit the commit message.

## Command: edit - Modify Commit

```bash
pick abc1234 Add login feature
edit def5678 Add tests  # Stop here to make changes
```

Git stops at that commit:

```bash
# Make changes
vim test.js
git add test.js

# Amend the commit
git commit --amend

# Continue rebase
git rebase --continue
```

### Splitting a Commit

```bash
edit def5678 Add multiple features

# Git stops - reset but keep changes
git reset HEAD^

# Commit separately
git add feature1.js
git commit -m "Add feature 1"

git add feature2.js
git commit -m "Add feature 2"

# Continue
git rebase --continue
```

## Command: squash - Combine Commits

```bash
pick abc1234 Add login feature
squash def5678 Fix typo
squash ghi9012 Fix another typo
```

Combines all three into one commit:

1. Merges the changes
2. Opens editor with all three messages
3. Lets you write new combined message

**Result:** One commit with all changes.

## Command: fixup - Squash Without Message

Like squash, but discards the commit message.

```bash
pick abc1234 Add login feature
fixup def5678 Fix typo
fixup ghi9012 Fix another typo
```

**Result:** One commit with message from `abc1234`, others discarded.

## Command: drop - Remove Commit

```bash
pick abc1234 Add login feature
drop def5678 Bad commit  # Remove this
pick ghi9012 Add tests
```

Alternative - just delete the line:

```bash
pick abc1234 Add login feature
# def5678 line deleted
pick ghi9012 Add tests
```

## Command: exec - Run Shell Command

```bash
pick abc1234 Add login feature
exec npm test  # Run tests after this
pick def5678 Add another feature
exec npm test  # Run tests again
```

If command fails, rebase stops. Useful for verifying builds/tests.

## Reordering Commits

Just change the order:

```bash
# Original:
pick abc1234 Commit A
pick def5678 Commit B
pick ghi9012 Commit C

# Reorder to:
pick ghi9012 Commit C  # Now first
pick abc1234 Commit A
pick def5678 Commit B
```

⚠️ **Watch out** - reordering can cause conflicts if commits depend on each other.

## Autosquash Workflow

Use during development:

```bash
# Initial commit
git commit -m "Add login feature"  # abc1234

# Continue working...
git commit -m "Add tests"

# Find bug in original commit
git add fix.js
git commit --fixup abc1234
# Creates "fixup! Add login feature"

# More work...
git commit -m "Add documentation"

# Another fix for original
git add another-fix.js
git commit --fixup abc1234

# Clean up with autosquash
git rebase -i --autosquash HEAD~6

# Git automatically arranges:
pick abc1234 Add login feature
fixup xxx111 fixup! Add login feature
fixup yyy222 fixup! Add login feature
pick zzz333 Add tests
pick aaa444 Add documentation
```

## Real-World Example

Messy commits before PR:

```bash
git log --oneline
# jkl3456 Add feature - final
# ghi9012 WIP
# def5678 Fix bug
# abc1234 Add feature - initial
# xyz7890 More WIP
```

Clean it up:

```bash
git rebase -i HEAD~5

# Change to:
pick abc1234 Add feature - initial
squash xyz7890 More WIP
squash ghi9012 WIP
squash jkl3456 Add feature - final
pick def5678 Fix bug

# Result: 2 clean commits
# - "Add login feature" (combined)
# - "Fix bug"
```

## Abort or Continue

```bash
# If things go wrong
git rebase --abort  # Cancel everything

# After resolving conflicts
git add .
git rebase --continue

# Skip a commit causing problems
git rebase --skip
```

## Practice Exercise

```bash
# Create messy commits
echo "v1" > file.txt
git add file.txt
git commit -m "Add file"

echo "v2" >> file.txt
git commit -am "WIP"

echo "v3" >> file.txt
git commit -am "More WIP"

echo "v4" >> file.txt
git commit -am "Final version"

# Clean up
git rebase -i HEAD~3

# Squash the WIP commits:
pick <first> Add file
squash <second> WIP
squash <third> More WIP
squash <fourth> Final version

# Result: One clean commit!
```

## Common Interactive Rebase Patterns

### Pattern 1: Squash all WIP commits

```bash
pick abc1234 Start feature
fixup def5678 WIP
fixup ghi9012 WIP
fixup jkl3456 WIP
pick mno7890 Final commit
```

### Pattern 2: Fix commit messages

```bash
reword abc1234 typo in mesage
pick def5678 Good message
reword ghi9012 another typo
```

### Pattern 3: Remove debug commits

```bash
pick abc1234 Add feature
drop def5678 Debug logging
pick ghi9012 Add tests
drop jkl3456 More debug
```

### Pattern 4: Reorder for logical flow

```bash
# Move test commit after feature
pick abc1234 Add feature
pick ghi9012 Add tests  # Was 3rd
pick def5678 Fix bug    # Was 2nd
```