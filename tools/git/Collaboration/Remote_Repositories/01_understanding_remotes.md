# 5.1 Understanding Remotes

## What is a Remote?

A "remote" is just a URL pointing to another git repository.

```bash
# See your remotes
git remote -v

# If you cloned from GitHub, you'll see:
# origin  [https://github.com/user/repo.git](https://github.com/user/repo.git) (fetch)
# origin  [https://github.com/user/repo.git](https://github.com/user/repo.git) (push)
```

**"origin"** is the default name for the remote you cloned from (like "the original source").

## Adding Remotes

```bash
# Add a new remote
git remote add upstream [https://github.com/original/repo.git](https://github.com/original/repo.git)

# View all remotes
git remote -v
# origin    [https://github.com/yourfork/repo.git](https://github.com/yourfork/repo.git) (fetch)
# origin    [https://github.com/yourfork/repo.git](https://github.com/yourfork/repo.git) (push)
# upstream  [https://github.com/original/repo.git](https://github.com/original/repo.git) (fetch)
# upstream  [https://github.com/original/repo.git](https://github.com/original/repo.git) (push)
```

## Managing Remotes

```bash
# Show detailed info about a remote
git remote show origin

# Rename a remote
git remote rename origin github

# Remove a remote
git remote remove upstream

# Change a remote's URL
git remote set-url origin [https://new-url.git](https://new-url.git)
```

## Remote-Tracking Branches

When you have remotes, you get **remote-tracking branches**:

```bash
git branch -a
# * main                          (your local branch)
#   remotes/origin/main           (tracks GitHub's main)
#   remotes/origin/feature-x      (another branch on GitHub)
```

<aside>
💡

**Key concept:** `origin/main` is a **local copy** of what `main` looked like on the remote last time you fetched. It's not "live" - it's a snapshot.

</aside>

## Viewing Remote Branches

```bash
# List only remote branches
git branch -r

# List all branches (local + remote)
git branch -a

# View commits on a remote branch
git log origin/main

# Compare local vs remote
git diff main origin/main
```