# 4.3 Merging Branches

Once your feature is done, you want to bring it back to `main`. This is **merging**.

## Basic Merge Command

```bash
# Go to the branch you want to merge INTO (usually main)
git checkout main

# Merge the feature branch INTO main
git merge feature-login
```

## Two Types of Merges

### 1. Fast-Forward Merge (Simple Case)

When `main` hasn't changed since you branched:

```
Before merge:          After merge:

main                   main, feature
 |                       |
 v                       v
 A                       A
 |                       |
 B                       B
 |                       |
feature                  C
 v
 C
```

Git just moves `main`'s pointer forward. No merge commit needed!

```bash
# Example
git checkout main
git checkout -b feature1
echo "feature 1" > f1.txt
git add f1.txt
git commit -m "Add feature 1"

git checkout main
git merge feature1
# Output: "Fast-forward"
```

### 2. Three-Way Merge (Diverged Branches)

When both `main` and `feature` have new commits:

```
Before merge:

    A -- B -- D     (main had commit D)
     \      
      C           (feature had commit C)
       ^
     feature

After merge:

    A -- B -- D -- M    (M is the merge commit)
     \           /
      C ---------
```

Git creates a **new merge commit** that combines both branches.

```bash
# Example of divergence
git checkout main
echo "main work" > main.txt
git add main.txt
git commit -m "Work on main"

git checkout -b feature2
echo "feature 2" > f2.txt
git add f2.txt
git commit -m "Add feature 2"

git checkout main
git merge feature2
# A merge commit is created!
```

## Hands-On Exercise

```bash
# Start clean
git checkout main

# Create and work on feature1 (fast-forward)
git checkout -b feature1
echo "feature 1" > f1.txt
git add f1.txt
git commit -m "Add feature 1"

git checkout main
git merge feature1
# Should see: "Fast-forward"

# Create feature2 that diverges
git checkout -b feature2
echo "feature 2" > f2.txt
git add f2.txt
git commit -m "Add feature 2"

# Work on main too (creates divergence)
git checkout main
echo "main work" > main.txt
git add main.txt
git commit -m "Work on main"

# Now merge feature2 (three-way merge!)
git merge feature2
# A merge commit is created

# Look at the history
git log --oneline --graph --all
```

## Viewing Merge Commits

```bash
# Show the merge commit
git log --oneline -1

# Show its parents
git show HEAD
# You'll see "Merge: abc1234 def5678" - the two parent commits!
```

## Perforce Comparison

**Perforce integrations** (`p4 integrate`, `p4 resolve`) are like merges, but:

- **In Perforce:** Heavy, requires server, creates copies
- **In Git:** Instant, local, just moves pointers