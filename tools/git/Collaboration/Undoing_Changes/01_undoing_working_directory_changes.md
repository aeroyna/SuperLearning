# 6.1 Undoing Working Directory Changes

## Scenario: Discard Uncommitted Changes

You edited a file but haven't staged it yet. You want to discard the changes.

```bash
# Edit a file
echo "bad changes" >> file.txt
git status
# Changes not staged: modified: file.txt

# Undo the changes (restore from staging/last commit)
git restore file.txt
# OR (old way):
git checkout -- file.txt

# file.txt is back to last committed state
```

## Restore All Modified Files

```bash
# Discard all changes in working directory
git restore .

# Restore specific directory
git restore src/

# Restore specific file
git restore path/to/file.txt
```

<aside>
⚠️

**Warning:** This permanently discards your changes! They're gone forever. There's no way to recover uncommitted, untracked changes.

</aside>

## Checking Before Restore

```bash
# See what you're about to discard
git diff file.txt

# Make sure you want to proceed
git restore file.txt
```

## Restore vs Checkout (Old Way)

The old way used `checkout`, which was confusing because it did multiple things:

```bash
# Old way (still works)
git checkout -- file.txt

# New way (clearer intent)
git restore file.txt
```

Both do the same thing, but `restore` is more explicit about undoing changes.