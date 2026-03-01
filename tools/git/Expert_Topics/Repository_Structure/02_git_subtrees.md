# 11.2 Git Subtrees

## What are Subtrees?

Subtrees merge external repos into your main repo as a subdirectory. Unlike submodules, the code becomes part of your repo.

## Subtree vs Submodule

| Feature | Submodule | Subtree |
| --- | --- | --- |
| Storage | Separate repo reference | Merged into main repo |
| Cloning | Requires --recurse-submodules | Just works |
| Workflow | More complex | Simpler |
| Size | Smaller | Larger |
| Updates | Manual | Manual but easier |
| History | Separate | Can be merged or squashed |

## Adding a Subtree

```bash
# Add remote
git remote add lib-remote [https://github.com/user/library.git](https://github.com/user/library.git)

# Add subtree (pulls entire history by default)
git subtree add --prefix=lib/external lib-remote main --squash

# --squash: Only keep one commit instead of full history
# --prefix: Where to put the subtree
```

<aside>
💡

**Tip:** Always use `--squash` unless you need full history. It keeps your repo cleaner.

</aside>

## Pulling Subtree Updates

```bash
# Pull updates from subtree
git subtree pull --prefix=lib/external lib-remote main --squash

# This merges changes from the external repo
```

## Pushing Changes to Subtree

```bash
# Make changes in subtree directory
vim lib/external/file.txt
git commit -am "Update library"

# Push changes back to subtree repo
git subtree push --prefix=lib/external lib-remote feature-branch
```

## Extracting a Subtree

```bash
# Split subtree into its own branch
git subtree split --prefix=lib/external -b lib-only

# Push to new repo
git push [git@github.com](mailto:git@github.com):user/new-library.git lib-only:main
```

## Complete Subtree Workflow

```bash
# Initial setup
git remote add vendor-lib [https://github.com/vendor/lib.git](https://github.com/vendor/lib.git)
git subtree add --prefix=vendor/lib vendor-lib main --squash
git push

# Later, update the library
git subtree pull --prefix=vendor/lib vendor-lib main --squash
git push

# Make local changes
vim vendor/lib/config.js
git commit -am "Customize library config"
git push

# Contribute back (if you have permissions)
git subtree push --prefix=vendor/lib vendor-lib feature-branch
```

## Subtree with Multiple Libraries

```bash
# Add multiple subtrees
git remote add lib-a [https://github.com/vendor/lib-a.git](https://github.com/vendor/lib-a.git)
git subtree add --prefix=vendor/lib-a lib-a main --squash

git remote add lib-b [https://github.com/vendor/lib-b.git](https://github.com/vendor/lib-b.git)
git subtree add --prefix=vendor/lib-b lib-b main --squash

# Update all
git subtree pull --prefix=vendor/lib-a lib-a main --squash
git subtree pull --prefix=vendor/lib-b lib-b main --squash
```

## Subtree Best Practices

1. **Always use --squash** - Keeps history clean
2. **Document remotes** - Keep track of subtree sources
3. **Use consistent prefixes** - vendor/, lib/, external/
4. **Pull before push** - Stay in sync with upstream
5. **Create aliases** - Simplify common commands

## Subtree Aliases

```bash
# Add to ~/.gitconfig
[alias]
  # Add subtree
  sba = "!f() { git subtree add --prefix $2 $1 main --squash; }; f"
  # Pull subtree
  sbpl = "!f() { git subtree pull --prefix $2 $1 main --squash; }; f"
  # Push subtree
  sbps = "!f() { git subtree push --prefix $2 $1 main; }; f"

# Usage:
# git sba lib-remote vendor/lib
# git sbpl lib-remote vendor/lib
# git sbps lib-remote vendor/lib
```

## Advantages of Subtrees

✅ **Advantages:**

- Simpler workflow than submodules
- No special clone commands needed
- All code in one repo
- Team doesn't need to learn new commands
- Works with standard Git commands
- No .gitmodules complexity

## Disadvantages of Subtrees

❌ **Disadvantages:**

- Larger repository size
- History can get messy without --squash
- Harder to see what came from where
- Pull/push commands are verbose
- Can't easily switch versions

## When to Use Subtrees

✅ **Use subtrees when:**

- You want simpler clone/pull workflow
- External code doesn't change often
- You're ok with larger repo size
- Team isn't familiar with submodules
- You need all code in one place
- Simplicity over precision

❌ **Use submodules when:**

- External code changes frequently
- You need smaller repo size
- You need precise version control
- Multiple projects share dependency
- You want clearer separation

## Migrating from Submodule to Subtree

```bash
# Remove submodule
git submodule deinit lib/external
git rm lib/external
rm -rf .git/modules/lib/external
git commit -m "Remove submodule"

# Add as subtree
git remote add lib-remote [https://github.com/user/library.git](https://github.com/user/library.git)
git subtree add --prefix=lib/external lib-remote main --squash
git commit -m "Add as subtree"
```

## Subtree Troubleshooting

### Problem: Conflicts during pull

```bash
# Resolve conflicts normally
vim conflicted-file.txt
git add conflicted-file.txt
git commit
```

### Problem: Lost track of remote

```bash
# List remotes
git remote -v

# Re-add if needed
git remote add lib-remote [https://github.com/user/library.git](https://github.com/user/library.git)
```

### Problem: Accidentally committed to subtree

```bash
# Split and push to upstream
git subtree push --prefix=lib/external lib-remote feature-branch
```