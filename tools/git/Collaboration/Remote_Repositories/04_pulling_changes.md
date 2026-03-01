# 5.4 Pulling Changes

## Pull: Fetch + Merge

```bash
# Pull = fetch + merge (like p4 sync)
git pull origin main
```

## What `git pull` Does

1. **Fetches** new commits from `origin`
2. **Merges** `origin/main` into your local `main`

```
Before pull:
Local:        A -- B -- C (main)
Remote:       A -- B -- C -- D -- E (origin/main)

After pull:
Local:        A -- B -- C ----------- M (main)
                         \           /
                          D ------- E (origin/main)
```

## Fetch vs Pull

### Fetch: Download Only (Safe)

```bash
# Fetch: Download commits but don't merge
git fetch origin

# Now you can see what changed:
git log origin/main
git log --oneline main..origin/main  # Show commits you don't have

# Compare differences
git diff main origin/main

# Decide to merge later:
git merge origin/main
```

### Pull: Download + Merge (Convenient)

```bash
# Pull does fetch + merge in one step
git pull origin main
```

<aside>
💡

**Think of it as:**

- `fetch` = download (safe, doesn't change your files)
- `merge` = integrate (changes your files)
- `pull` = `fetch` + `merge`
</aside>

## Pull with Rebase

```bash
# Pull with rebase instead of merge
git pull --rebase origin main

# This creates a linear history instead of merge commits
```

**Regular pull:**

```
A -- B -- C ----------- M (merge commit)
         \           /
          D ------- E
```

**Pull with rebase:**

```
A -- B -- D -- E -- C' (your commit replayed on top)
```

## Handling Pull Conflicts

```bash
# Pull and get conflicts
git pull origin main
# CONFLICT: Merge conflict in file.txt

# Resolve conflicts
# Edit file.txt, remove conflict markers
git add file.txt
git commit  # Complete the merge
```

## Checking Before Pull

```bash
# Fetch first to see what's new
git fetch origin

# Check how many commits behind you are
git status
# "Your branch is behind 'origin/main' by 3 commits"

# See what commits you'll get
git log --oneline main..origin/main

# See what changed
git diff main origin/main

# Now decide to pull
git pull origin main
```

## Setting Upstream for Pull

```bash
# Set upstream (do once)
git branch --set-upstream-to=origin/main main

# OR when pushing:
git push -u origin main

# Now you can just use:
git pull  # Knows to pull from origin/main
```