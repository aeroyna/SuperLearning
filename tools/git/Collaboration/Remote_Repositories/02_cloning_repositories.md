# 5.2 Cloning Repositories

## Cloning a Repository

```bash
# Clone a repo from GitHub (like p4 sync for the first time)
git clone [https://github.com/username/repo.git](https://github.com/username/repo.git)

# Clone into a specific directory name
git clone [https://github.com/username/repo.git](https://github.com/username/repo.git) my-project

# Clone a specific branch
git clone -b develop [https://github.com/username/repo.git](https://github.com/username/repo.git)
```

## What Cloning Does

When you clone, git performs several operations:

1. Creates a new directory with the repo name
2. Initializes a git repo inside it
3. Adds "origin" as a remote pointing to the source URL
4. Fetches all data from origin (all commits, branches, history)
5. Checks out the default branch (usually `main` or `master`)

```bash
cd repo
git remote -v  # You'll see "origin" automatically configured
git branch -a  # See all local and remote branches
git log        # Full history is available locally!
```

## After Cloning

```bash
# You're on the default branch
git branch
# * main

# You have a complete copy of the repository
git log --oneline
# Shows full commit history

# Remote is set up
git remote -v
# origin  [https://github.com/username/repo.git](https://github.com/username/repo.git) (fetch)
# origin  [https://github.com/username/repo.git](https://github.com/username/repo.git) (push)
```

## Clone vs Download ZIP

**Download ZIP from GitHub:**

- Just the files, no git history
- No connection to the remote
- Can't push/pull changes

**Git clone:**

- Full repository with all history
- Connected to remote via "origin"
- Can push/pull changes
- Can see all branches

## Shallow Clone (Advanced)

For very large repos, you can clone only recent history:

```bash
# Clone only the last 10 commits
git clone --depth 10 [https://github.com/username/repo.git](https://github.com/username/repo.git)

# Clone only a single branch
git clone --single-branch --branch main [https://github.com/username/repo.git](https://github.com/username/repo.git)
```