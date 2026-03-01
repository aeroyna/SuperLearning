# 6.2 Unstaging Files

## Scenario: Unstage Without Losing Changes

You staged a file but want to unstage it. The changes remain in your working directory.

```bash
# Stage a file
echo "new content" > file.txt
git add file.txt
git status
# Changes to be committed: modified: file.txt

# Unstage it (keep the changes in working directory)
git restore --staged file.txt
# OR (old way):
git reset HEAD file.txt

git status
# Changes not staged: modified: file.txt
# The file still has your changes, just not staged anymore
```

## Unstage Multiple Files

```bash
# Unstage all files
git restore --staged .

# Unstage specific files
git restore --staged file1.txt file2.txt

# Unstage a directory
git restore --staged src/
```

## Understanding the Staging Area

```
Working Directory    Staging Area    Repository
     (edit)    -->      (add)    -->   (commit)
                    <-- (restore --staged)
```

`git restore --staged` moves changes from staging back to working directory.

## Common Pattern: Selective Staging

```bash
# You changed 5 files but want to commit them separately
vim file1.txt file2.txt file3.txt file4.txt file5.txt

# Stage everything
git add .

# Oops, file5.txt shouldn't be in this commit
git restore --staged file5.txt

# Commit the other 4
git commit -m "Update files 1-4"

# Stage and commit file5 separately
git add file5.txt
git commit -m "Update file 5"
```