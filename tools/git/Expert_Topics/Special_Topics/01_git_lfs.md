# 13.1 Git LFS

## What is Git LFS?

Git LFS (Large File Storage) handles large files efficiently by storing pointers in Git and actual files on a separate server.

**Problem with large files in Git:**

- Git stores full history of every file
- Large binary files make repos huge
- Clone/fetch/push become very slow
- Binary diffs don't compress well

**LFS Solution:**

- Store pointers in Git repo (tiny)
- Store actual files on LFS server
- Download files only when needed

## Installing Git LFS

```bash
# Mac
brew install git-lfs

# Ubuntu/Debian
sudo apt-get install git-lfs

# Windows
# Download from [https://git-lfs.github.com/](https://git-lfs.github.com/)

# Initialize LFS (one-time setup)
git lfs install
```

## Tracking Files with LFS

```bash
# Track specific file types
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"
git lfs track "*.bin"

# Track specific files
git lfs track "large-dataset.csv"

# Track files in directory
git lfs track "assets/**"

# This creates/updates .gitattributes
git add .gitattributes
git commit -m "Configure LFS tracking"
```

## Viewing Tracked Files

```bash
# List tracked patterns
git lfs track

# List LFS files in repo
git lfs ls-files

# Show file status
git lfs status

# Show LFS file info
git lfs ls-files --size
```

## Working with LFS Files

```bash
# Add large file (automatically stored in LFS)
git add [large-video.mp](http://large-video.mp)4
git commit -m "Add video"
git push

# Clone repo with LFS files
git clone [https://github.com/user/repo.git](https://github.com/user/repo.git)
cd repo
git lfs pull  # Download LFS files

# Or clone with LFS in one step
git lfs clone [https://github.com/user/repo.git](https://github.com/user/repo.git)
```

## LFS File Pointers

What Git actually stores (not the file itself):

```
version [https://git-lfs.github.com/spec/v1](https://git-lfs.github.com/spec/v1)
oid sha256:4d7a214614ab2935c943f9e0ff69d22eadbb8f32b1258daaa5e2ca24d17e2393
size 12345678
```

The pointer is tiny (< 1KB), actual file stored on LFS server.

## Managing LFS Storage

```bash
# See how much storage is used
git lfs ls-files --size

# Fetch only specific files
git lfs fetch --include="*.psd"

# Exclude files from fetch
git lfs fetch --exclude="*.mp4"

# Prune old LFS files (free up space)
git lfs prune

# Fetch LFS files for specific commit
git lfs fetch origin abc1234
```

## Migrating Existing Files to LFS

```bash
# Migrate existing large files to LFS
git lfs migrate import --include="*.zip" --everything

# Migrate specific files
git lfs migrate import --include="large-file.bin"

# This rewrites history, so coordinate with team!
git push --force-with-lease origin --all
```

## LFS Best Practices

1. **Track early** - Add .gitattributes before committing large files
2. **Be specific** - Only track files that need LFS
3. **Document** - Explain LFS setup in README
4. **Server setup** - Ensure LFS server is configured
5. **Team coordination** - Everyone needs LFS installed
6. **Monitor usage** - Watch storage/bandwidth limits
7. **Prune regularly** - Clean up old LFS objects

## When to Use LFS

✅ **Use LFS for:**

- Large binary files (videos, images, datasets)
- Files that change frequently
- Build artifacts, compiled binaries
- Media assets for projects
- Machine learning models
- Game assets

❌ **Don't use LFS for:**

- Text files (compress well in Git)
- Small files (< 1MB)
- Files that rarely change
- Source code