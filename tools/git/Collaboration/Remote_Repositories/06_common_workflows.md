# 5.6 Common Workflows

## Daily Development Workflow

```bash
# Start of day: get latest from team
git checkout main
git pull origin main

# Work on a feature
git checkout -b my-feature
echo "feature work" > feature.txt
git add feature.txt
git commit -m "Add feature"

# Push your feature
git push -u origin my-feature

# Team reviews on GitHub (Pull Request)
# After approval, merge on GitHub

# Update your local main
git checkout main
git pull origin main

# Delete local feature branch
git branch -d my-feature

# Delete remote feature branch (if not auto-deleted)
git push origin --delete my-feature
```

## Syncing with Upstream (Fork Workflow)

```bash
# You forked someone's repo
# Keep your fork in sync with the original

# Add upstream remote (do once)
git remote add upstream [https://github.com/original/repo.git](https://github.com/original/repo.git)

# Regular sync:
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

## Feature Branch Workflow

```bash
# 1. Start from updated main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/user-authentication

# 3. Work and commit
git add .
git commit -m "Implement user login"
git add .
git commit -m "Add password validation"

# 4. Push to remote
git push -u origin feature/user-authentication

# 5. Open Pull Request on GitHub

# 6. Address review comments
git add .
git commit -m "Fix review comments"
git push  # Updates the PR automatically

# 7. After merge, clean up
git checkout main
git pull origin main
git branch -d feature/user-authentication
```

## Handling Diverged Branches

```bash
# Your branch and origin/main have diverged
git status
# "Your branch and 'origin/main' have diverged,
#  and have 3 and 5 different commits each"

# Option 1: Merge (creates merge commit)
git pull origin main

# Option 2: Rebase (linear history)
git pull --rebase origin main
```

## Quick Reference

```bash
# Cloning
git clone <url>                    # Clone repository

# Remotes
git remote -v                      # List remotes
git remote add <name> <url>        # Add remote
git remote remove <name>           # Remove remote

# Fetching
git fetch origin                   # Fetch from origin
git fetch origin main              # Fetch specific branch

# Pulling (fetch + merge)
git pull origin main               # Fetch and merge
git pull --rebase origin main      # Fetch and rebase
git pull                           # If upstream set

# Pushing
git push origin main               # Push to origin/main
git push -u origin main            # Set upstream
git push                           # If upstream set
git push origin --delete branch    # Delete remote branch

# Branches
git branch -r                      # List remote branches
git branch -a                      # List all branches
git branch -vv                     # Show tracking info
git checkout -b feat origin/feat   # Create local from remote
```

## Perforce Translation

- `p4 sync` → `git pull origin main`
- `p4 submit` → `git push origin main`
- Creating workspace → `git clone <url>`
- Checking server status → `git fetch` then `git status`
- `p4 changes` → `git log origin/main`

## Practice Exercise

### With GitHub Account:

```bash
# 1. Create a test repo on GitHub ("New repository")
# 2. Clone it
git clone [https://github.com/yourusername/test-repo.git](https://github.com/yourusername/test-repo.git)
cd test-repo

# 3. Make a commit
echo "Hello from local" > [README.md](http://README.md)
git add [README.md](http://README.md)
git commit -m "Initial commit"

# 4. Push it
git push origin main

# 5. Check GitHub - you should see your commit!

# 6. Edit [README.md](http://README.md) on GitHub's web interface
# 7. Pull it down
git pull origin main

# 8. Check your local file - it has the web changes!
```

### Without GitHub (Simulate with Local Repos):

```bash
# Create a "remote" repo (bare repository)
mkdir remote-repo
cd remote-repo
git init --bare  # A bare repo acts like a server

# Create your "local" repo
cd ..
mkdir local-repo
cd local-repo
git init
git remote add origin ../remote-repo

# Make commits and push
echo "test" > file.txt
git add file.txt
git commit -m "Test commit"
git push -u origin main

# Simulate another developer
cd ..
git clone remote-repo local-repo-2
cd local-repo-2
git log  # See the commit from local-repo!

# Make changes in local-repo-2
echo "from dev 2" > dev2.txt
git add dev2.txt
git commit -m "Dev 2 commit"
git push origin main

# Pull in local-repo
cd ../local-repo
git pull origin main
ls  # You'll see dev2.txt!
```