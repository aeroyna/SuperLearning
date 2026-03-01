# 4.5 Branch Management

## Deleting Branches

Once a feature is merged, you don't need the branch anymore.

### Safe Delete (Merged Branches Only)

```bash
# Delete a branch that's been merged
git branch -d feature1

# Git will warn you if it's not merged:
# "error: The branch 'feature1' is not fully merged"
```

### Force Delete

```bash
# Force delete even if not merged (be careful!)
git branch -D feature1
```

## Listing Branches

```bash
# List local branches
git branch

# List with last commit info
git branch -v

# See which branches are merged into current branch
git branch --merged

# See which branches are NOT merged
git branch --no-merged
```

## Renaming Branches

```bash
# Rename current branch
git branch -m new-name

# Rename a different branch
git branch -m old-name new-name
```

## Quick Reference: All Branch Commands

```bash
# Creating and switching
git branch feature              # Create branch
git checkout feature            # Switch to branch
git checkout -b feature         # Create and switch
git switch feature              # Switch (newer)
git switch -c feature           # Create and switch (newer)

# Viewing
git branch                      # List branches
git branch -v                   # List with commit info
git branch --merged             # List merged branches
git branch --no-merged          # List unmerged branches

# Merging
git merge feature               # Merge feature into current branch
git merge --abort               # Cancel merge

# Deleting
git branch -d feature           # Delete merged branch
git branch -D feature           # Force delete

# Renaming
git branch -m new-name          # Rename current branch
git branch -m old new           # Rename specific branch
```

## Practice Scenario: Creating a Complex Branch Structure

Try creating this branching structure:

```
main -- A -- B ----------- M1 ----------- M2
         \                /               /
          C -- D (feat1) /               /
           \                            /
            E -- F (feat2) ------------
```

### Steps:

```bash
# 1. Start with 2 commits on main (A, B)
git checkout main
echo "A" > file.txt
git add file.txt
git commit -m "Commit A"
echo "B" >> file.txt
git commit -am "Commit B"

# 2. Branch to feat1 from A, make 2 commits (C, D)
git checkout HEAD~1  # Go back to A
git checkout -b feat1
echo "C" > feat1.txt
git add feat1.txt
git commit -m "Commit C"
echo "D" >> feat1.txt
git commit -am "Commit D"

# 3. Branch to feat2 from B, make 2 commits (E, F)
git checkout main  # Back to B
git checkout -b feat2
echo "E" > feat2.txt
git add feat2.txt
git commit -m "Commit E"
echo "F" >> feat2.txt
git commit -am "Commit F"

# 4. Merge feat1 into main (M1)
git checkout main
git merge feat1 -m "Merge M1"

# 5. Merge feat2 into main (M2)
git merge feat2 -m "Merge M2"

# View the result
git log --oneline --graph --all
```

## Clean Up After Practice

```bash
# Delete the feature branches
git branch -d feat1
git branch -d feat2

# Verify they're gone
git branch
```