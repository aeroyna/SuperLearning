# 11.1 Git Submodules

## What are Submodules?

Submodules allow you to keep a Git repository as a subdirectory of another Git repository.

**Use cases:**

- Include a library in your project
- Share code between multiple projects
- Vendor dependencies
- Keep main repo clean while including external code

**Example structure:**

```
my-project/
├── src/
├── lib/
│   └── external-lib/  ← Submodule (separate repo)
└── .gitmodules
```

## Adding a Submodule

```bash
# Add submodule
git submodule add [https://github.com/user/repo.git](https://github.com/user/repo.git) lib/external

# This creates:
# - lib/external/ directory with the repo content
# - .gitmodules file (tracks submodule info)
# - Entry in .git/config

# Commit the submodule
git commit -m "Add external library as submodule"
```

## The .gitmodules File

```bash
# .gitmodules
[submodule "lib/external"]
    path = lib/external
    url = [https://github.com/user/repo.git](https://github.com/user/repo.git)
    branch = main
```

This file tracks:

- Submodule path in your repo
- URL of the submodule repo
- Branch to track (optional)

## Cloning a Repo with Submodules

```bash
# Option 1: Clone and initialize submodules separately
git clone [https://github.com/user/main-repo.git](https://github.com/user/main-repo.git)
cd main-repo
git submodule init
git submodule update

# Option 2: Clone with submodules in one command
git clone --recursive [https://github.com/user/main-repo.git](https://github.com/user/main-repo.git)

# Option 3: Clone and update submodules recursively
git clone --recurse-submodules [https://github.com/user/main-repo.git](https://github.com/user/main-repo.git)
```

<aside>
💡

**Best practice:** Always use `--recurse-submodules` when cloning repos with submodules.

</aside>

## Updating Submodules

```bash
# Update all submodules to latest commit on tracked branch
git submodule update --remote

# Update specific submodule
git submodule update --remote lib/external

# Update and merge (instead of checkout)
git submodule update --remote --merge

# Update and rebase
git submodule update --remote --rebase
```

## Working with Submodules

```bash
# Enter submodule directory
cd lib/external

# It's a normal git repo
git status
git log
git checkout -b feature

# Make changes
vim file.txt
git commit -am "Update library"

# Push changes to submodule repo
git push origin feature

# Go back to main repo
cd ../..

# Main repo sees submodule changed
git status
# modified: lib/external (new commits)

# Commit the submodule update in main repo
git add lib/external
git commit -m "Update external library to latest"
git push
```

## Checking Submodule Status

```bash
# List all submodules
git submodule

# Show submodule status
git submodule status

# Output:
#  abc1234... lib/external (v1.2.0)
#  ^        ^             ^
#  |        |             └─ tag/branch
#  |        └─────────────── path
#  └──────────────────────── commit hash

# Check for changes
git diff --submodule
```

## Submodule Foreach

```bash
# Run command in all submodules
git submodule foreach 'git pull origin main'

# Check status of all submodules
git submodule foreach 'git status'

# Show branches in all submodules
git submodule foreach 'git branch -a'
```

## Removing a Submodule

```bash
# 1. Deinitialize submodule
git submodule deinit lib/external

# 2. Remove from git tracking
git rm lib/external

# 3. Remove from .git/modules
rm -rf .git/modules/lib/external

# 4. Commit
git commit -m "Remove external library submodule"
```

## Submodule Pitfalls

### Problem 1: Detached HEAD

```bash
# Submodules are checked out in detached HEAD state by default
cd lib/external
git status
# HEAD detached at abc1234

# Solution: Create a branch
git checkout -b work
```

### Problem 2: Forgetting to Push Submodule

```bash
# Changed submodule but forgot to push
cd lib/external
git push  # Must push submodule first!

cd ../..
git push  # Then push main repo
```

### Problem 3: Submodule Not Updated

```bash
# Pulled main repo but submodule not updated
git pull
git submodule update --init --recursive
```

## Submodule Best Practices

1. **Always use --recurse-submodules when cloning**
2. **Push submodule changes before pushing main repo**
3. **Document submodule workflow in README**
4. **Use specific branches, not floating heads**
5. **Keep submodules shallow if possible**
6. **Check submodule status before committing**
7. **Configure Git to show submodule changes**

## Advanced Submodule Config

```bash
# Auto-update submodules on pull
git config submodule.recurse true

# Always show submodule summary
git config status.submoduleSummary true

# Use specific branch for submodule
git config -f .gitmodules submodule.lib/external.branch develop

# Shallow clone submodules (faster)
git clone --recurse-submodules --shallow-submodules
```

## Submodule Workflow Example

```bash
# Day 1: Add library
git submodule add [https://github.com/vendor/lib.git](https://github.com/vendor/lib.git) vendor/lib
git commit -m "Add vendor library"
git push

# Day 2: Update library
cd vendor/lib
git pull origin main
cd ../..
git add vendor/lib
git commit -m "Update vendor library to v2.0"
git push

# Day 3: Teammate clones
git clone --recurse-submodules [https://github.com/company/project.git](https://github.com/company/project.git)
cd project
# Submodules automatically set up!

# Day 4: Pull updates
git pull
git submodule update --remote
```

## When to Use Submodules

✅ **Use submodules when:**

- External code has its own repo
- Need precise version control
- Multiple projects share dependency
- Want smaller main repo
- External code changes frequently

❌ **Don't use submodules when:**

- Code doesn't have separate repo
- Team unfamiliar with Git
- Simple copy would suffice
- Need simpler workflow