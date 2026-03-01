# 4.2 Understanding Branch Behavior

## How Branches Affect Your Working Directory

When you switch branches, your files change to match that branch's state.

## Demonstration

```bash
# Create a branch with a new file
git checkout -b test-branch
echo "new feature" > feature.txt
git add feature.txt
git commit -m "Add feature"

# feature.txt exists here
ls  # You'll see feature.txt

# Switch back to main
git checkout main
ls  # feature.txt is GONE!

# Switch back to test-branch
git checkout test-branch
ls  # feature.txt is BACK!
```

## Key Insight

<aside>
🎯

Your working directory instantly reflects whichever branch you're on. Git swaps out files to match that branch's state.

**This is magic compared to Perforce!** No copying directories, no separate workspaces needed.

</aside>

## Each Branch is Independent

```
main:        A -- B
                   \
                    C -- D
                         ^
                    test-branch
```

- Each branch is an independent line of development
- You can switch between them instantly
- Your files change to match the branch

## Real-World Workflow

```bash
# You're on main, working on production code
git checkout main

# Suddenly: "We need to add login feature!"
git checkout -b feature-login
echo "login code" > login.js
git add login.js
git commit -m "Add login feature"

# Another urgent task: "Fix the homepage bug!"
git checkout -b bugfix-homepage
# Your login.js disappears! You're back to main's state
echo "fix" > homepage.html
git add homepage.html
git commit -m "Fix homepage bug"

# Now you have 3 branches:
git log --oneline --all --graph
```

You should see something like:

```
* xyz1234 (bugfix-homepage) Fix homepage bug
| * abc5678 (feature-login) Add login feature
|/
* def9012 (main) Original commit
```

See how they **diverged** from main? This is **branching**!