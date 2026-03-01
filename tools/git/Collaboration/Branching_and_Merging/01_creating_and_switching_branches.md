# 4.1 Creating and Switching Branches

## Creating Branches

```bash
# Create a new branch
git branch feature-login

# See your branches
git branch
# * main
#   feature-login

# Switch to the new branch
git checkout feature-login
# OR (newer command):
git switch feature-login

# Now you're on feature-login
git branch
#   main
# * feature-login
```

## Shortcut - Create and Switch in One Command

```bash
git checkout -b feature-payment
# OR
git switch -c feature-payment
```

## What `git checkout -b` Actually Does

<aside>
💡

**Important concept:** `git checkout -b feature-name` does TWO things:

1. **Creates** the branch `feature-name` pointing at the **current commit**
2. **Switches** to that branch (you stay at the same commit)

The branch is created at your current location and left there. When you switch back to another branch and that branch moves forward, that's when divergence happens.

</aside>

## Exercise: Create Your First Branch

```bash
git checkout -b test-branch
echo "new feature" > feature.txt
git add feature.txt
git commit -m "Add feature"

git log --oneline --all --graph
```

**What happened?**

- `test-branch` moved forward one commit
- `main` stayed where it was
- `test-branch` is now one commit ahead of `main`

## Quick Reference

```bash
git branch                    # List branches
git branch feature            # Create branch (but don't switch)
git checkout feature          # Switch to branch
git checkout -b feature       # Create and switch
git switch feature            # Switch (newer command)
git switch -c feature         # Create and switch (newer)
```