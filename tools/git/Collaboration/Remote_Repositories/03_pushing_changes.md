# 5.3 Pushing Changes

## Basic Push

```bash
# Make some commits locally
echo "new feature" > feature.txt
git add feature.txt
git commit -m "Add feature"

# Push to remote (like p4 submit)
git push origin main
# "Push the main branch to the origin remote"
```

## What Happens When You Push

1. Git sends your new commits to the remote (GitHub)
2. The remote's `main` branch moves forward to your latest commit
3. Your local `origin/main` tracking branch updates to match

```
Before push:
Local:        A -- B -- C (main)
Remote:       A -- B (origin/main)

After push:
Local:        A -- B -- C (main, origin/main)
Remote:       A -- B -- C
```

## Setting Upstream

```bash
# First push: set upstream tracking
git push -u origin main
# -u sets origin/main as the upstream for your local main

# Future pushes: just use
git push
# Git knows where to push!
```

## Pushing a New Branch

```bash
# Create a branch locally
git checkout -b feature-login
echo "login" > login.js
git commit -am "Add login"

# Push it to the remote
git push origin feature-login
# OR set as upstream:
git push -u origin feature-login

# Now others can see it on GitHub!
```

## Push Conflicts

```bash
# You try to push
git push origin main

# ERROR: "Updates were rejected because the remote contains work..."
# Someone else pushed while you were working!
```

<aside>
⚠️

**What happened:** The remote has commits you don't have. Git won't let you push because it would overwrite their work.

</aside>

**Solution: Pull first**

```bash
# Pull to get their changes (fetch + merge)
git pull origin main
# Fix any merge conflicts if needed

# Then push again:
git push origin main
```

This is like `p4 resolve` - you need to integrate others' changes before submitting.

## Force Push (Dangerous!)

```bash
# Force push (overwrites remote - USE WITH CAUTION!)
git push --force origin main
# OR safer version:
git push --force-with-lease origin main
```

<aside>
🚨

**Warning:** Force push rewrites history on the remote. Only use when:

- You're working alone on a branch
- You've coordinated with your team
- You understand the consequences

Never force push to shared branches like `main`!

</aside>

## Deleting Remote Branches

```bash
# Delete a branch from the remote
git push origin --delete feature-login

# Your local branch still exists
git branch -d feature-login  # Delete locally too
```