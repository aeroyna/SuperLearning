# 5.5 Working with Remote Branches

## Tracking Remote Branches

```bash
# See what branch tracks what
git branch -vv
# * main           abc1234 [origin/main] Latest commit
#   feature-login  def5678 [origin/feature-login: behind 2] Old commit
```

The `[origin/main]` shows your branch tracks that remote branch.

## Creating Local Branch from Remote

```bash
# Someone pushed a branch to GitHub, you want to work on it

# Option 1: Fetch and checkout
git fetch origin
git checkout -b feature-x origin/feature-x

# Option 2: Shortcut (auto-tracks if name matches)
git checkout feature-x
# Git sees origin/feature-x and creates local feature-x tracking it
```

## Pushing New Branches

```bash
# Create a local branch
git checkout -b my-feature
echo "work" > file.txt
git commit -am "Add feature"

# Push to remote and set upstream
git push -u origin my-feature

# Branch now exists on GitHub
# Other team members can check it out
```

## Checking Remote Branch Status

```bash
# Fetch latest remote info
git fetch origin

# See branch status
git branch -vv
# * main  abc1234 [origin/main: ahead 2, behind 1]
```

**ahead 2:** You have 2 commits not pushed

**behind 1:** Remote has 1 commit you don't have

## Deleting Remote Branches

```bash
# Delete remote branch
git push origin --delete feature-x

# Your local branch still exists
git branch
# * main
#   feature-x  (still here)

# Delete local branch too
git branch -d feature-x
```

## Pruning Stale Remote Branches

Someone deleted a branch on GitHub, but you still see it:

```bash
# See stale references
git branch -r
# origin/main
# origin/deleted-branch  (doesn't exist on remote anymore)

# Clean up stale references
git fetch origin --prune
# OR
git remote prune origin

# Now it's gone
git branch -r
# origin/main
```

## Working with Multiple Remotes

```bash
# Fork scenario: your fork + original repo
git remote -v
# origin    [https://github.com/you/repo.git](https://github.com/you/repo.git)
# upstream  [https://github.com/original/repo.git](https://github.com/original/repo.git)

# Fetch from upstream
git fetch upstream

# Merge upstream changes into your main
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

## Renaming Branches

```bash
# Rename local branch
git branch -m old-name new-name

# If you already pushed old-name, you need to:
# 1. Push the new name
git push origin new-name

# 2. Delete the old name from remote
git push origin --delete old-name

# 3. Update upstream tracking
git push origin -u new-name
```