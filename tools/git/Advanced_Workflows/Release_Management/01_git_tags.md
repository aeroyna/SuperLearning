# 9.1 Git Tags

## What are Tags?

Tags are pointers to specific commits, typically used to mark release versions. Unlike branches, tags don't move - they permanently point to one commit.

```bash
# Your commits
git log --oneline
# abc1234 Fix bug
# def5678 Add feature
# ghi9012 Release ready

# Tag this commit as v1.0.0
git tag v1.0.0 ghi9012
```

## Types of Tags

### 1. Lightweight Tags (Just a Pointer)

```bash
git tag v1.0.0
# Creates tag at current commit
```

### 2. Annotated Tags (Recommended)

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
# Stores: tagger name, email, date, message
```

<aside>
💡

**Best practice:** Always use annotated tags (`-a`) for releases. They include metadata about who tagged, when, and why.

</aside>

## Creating Tags

```bash
# Tag current commit (lightweight)
git tag v1.0.0

# Tag current commit (annotated)
git tag -a v1.0.0 -m "First release"

# Tag specific commit
git tag v1.0.0 abc1234

# Tag with annotation
git tag -a v1.0.0 abc1234 -m "Release message"
```

## Viewing Tags

```bash
# List all tags
git tag

# List tags matching pattern
git tag -l "v1.*"
# v1.0.0
# v1.0.1
# v1.1.0

# Show tag details
git show v1.0.0

# List tags with commit messages
git tag -n
# v1.0.0    Release version 1.0.0
# v1.0.1    Hotfix for bug

# See which commit a tag points to
git rev-list -n 1 v1.0.0
```

## Pushing Tags

<aside>
⚠️

**Important:** Tags are NOT pushed by default with `git push`!

</aside>

```bash
# Push specific tag
git push origin v1.0.0

# Push all tags
git push origin --tags

# Push annotated tags only (safer)
git push --follow-tags
```

## Checking Out Tags

```bash
# View code at a tag (detached HEAD state)
git checkout v1.0.0

# Create branch from tag
git checkout -b hotfix-1.0.1 v1.0.0
```

## Deleting Tags

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
# OR
git push origin :refs/tags/v1.0.0
```

## Tagging Old Commits

```bash
# Find commit to tag
git log --oneline

# Tag it
git tag -a v0.9.0 def5678 -m "Beta release"

# Push it
git push origin v0.9.0
```

## Semantic Versioning with Tags

```bash
# Major.Minor.Patch
git tag -a v1.0.0 -m "Major release"
git tag -a v1.0.1 -m "Patch: bug fixes"
git tag -a v1.1.0 -m "Minor: new features"
git tag -a v2.0.0 -m "Major: breaking changes"
```

**Semantic versioning:**

- **MAJOR** (v2.0.0): Breaking changes
- **MINOR** (v1.1.0): New features, backward compatible
- **PATCH** (v1.0.1): Bug fixes, backward compatible

## Release Workflow

```bash
# 1. Finish development
git checkout main
git pull origin main

# 2. Update version in code
vim package.json  # Update version
git commit -am "Bump version to 1.0.0"

# 3. Create tag
git tag -a v1.0.0 -m "Release 1.0.0: New features"

# 4. Push commit and tag
git push origin main
git push origin v1.0.0

# 5. Create release on GitHub using the tag
```

## Signed Tags (Security)

```bash
# Create GPG-signed tag
git tag -s v1.0.0 -m "Signed release"

# Verify signed tag
git tag -v v1.0.0
```

## Finding Tags

```bash
# Find tags containing a commit
git tag --contains abc1234

# Find tags pointing to a commit
git tag --points-at abc1234

# Sort tags by version
git tag -l --sort=-version:refname

# Show tags with dates
git tag -l --format='%(refname:short) %(creatordate:short)'
```

## Moving/Replacing Tags

```bash
# Move tag to different commit (force)
git tag -f v1.0.0 def5678

# Update on remote (force push)
git push origin v1.0.0 --force
```

<aside>
🚨

**Warning:** Moving tags on remote repositories can confuse other developers. Only do this for tags that haven't been widely distributed.

</aside>

## Tag Naming Conventions

```bash
# Releases
v1.0.0, v2.1.3

# Pre-releases
v1.0.0-alpha, v1.0.0-beta, v1.0.0-rc1

# Build metadata
v1.0.0+20240115, v1.0.0-beta+exp.sha.5114f85
```

## Tagging Best Practices

1. **Use annotated tags for releases** - Include message with details
2. **Follow semantic versioning** - MAJOR.MINOR.PATCH
3. **Include release notes** - What changed in this version
4. **Tag stable points** - Only tag tested, stable code
5. **Push tags explicitly** - Don't rely on `git push` alone
6. **Don't move published tags** - Once pushed, tags should be permanent
7. **Sign important releases** - Use GPG signatures for security

## Tags vs Branches

**Tags:**

- Point to specific commit, never move
- Mark release points
- Read-only references
- Good for: v1.0.0, v2.1.3

**Branches:**

- Move as new commits are added
- Active development
- Writable references
- Good for: feature-x, main, develop

## Tag Description Format

```bash
# Good tag message
git tag -a v1.5.0 -m "Release 1.5.0

New features:
- User authentication
- Dashboard redesign
- API v2 support

Bug fixes:
- Fixed login timeout
- Resolved memory leak

Breaking changes:
- Removed deprecated API endpoints"
```

## Listing Tags with Info

```bash
# Show tags with commit info
git tag -n5  # Show 5 lines of message

# Show tags with full details
for tag in $(git tag); do
  echo "Tag: $tag"
  git show $tag --quiet
  echo "---"
done
```

## Working with Remote Tags

```bash
# Fetch tags from remote
git fetch --tags

# Fetch specific tag
git fetch origin tag v1.0.0

# List remote tags
git ls-remote --tags origin

# Pull tags
git pull --tags
```