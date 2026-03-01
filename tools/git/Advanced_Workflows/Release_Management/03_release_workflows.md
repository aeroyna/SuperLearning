# 9.3 Release Workflows

## Basic Release Workflow

```bash
# 1. Finish development
git checkout main
git pull origin main

# 2. Update version in code
vim package.json  # Change "version": "1.4.2" to "1.5.0"
git commit -am "chore: bump version to 1.5.0"

# 3. Create tag
git tag -a v1.5.0 -m "Release v1.5.0

New features:
- User export functionality
- Performance improvements

Bug fixes:
- Fixed login timeout issue"

# 4. Push
git push origin main
git push origin v1.5.0

# 5. Create release on GitHub
# (Or use GitHub CLI: gh release create v1.5.0)
```

## Release Branch Workflow

For maintaining multiple versions:

```bash
# Develop on main
git checkout main

# Ready for release
git checkout -b release/1.5
git push origin release/1.5

# Tag the release
git tag -a v1.5.0 -m "Release 1.5.0"
git push origin v1.5.0

# Continue development on main (now 1.6.x)
git checkout main

# Hotfix for 1.5
git checkout release/1.5
git checkout -b hotfix/1.5.1
# Fix bug
git commit -am "fix: critical bug"
git checkout release/1.5
git merge hotfix/1.5.1
git tag -a v1.5.1 -m "Hotfix 1.5.1"
git push origin release/1.5
git push origin v1.5.1

# Merge hotfix back to main
git checkout main
git merge hotfix/1.5.1
```

## Changelog Generation

### Manual [CHANGELOG.md](http://CHANGELOG.md)

```markdown
# Changelog

## [1.5.0] - 2024-12-05

### Added
- User export functionality
- Bulk operations support

### Changed
- Improved performance for large datasets

### Fixed
- Login timeout issue
- Memory leak in background tasks

## [1.4.2] - 2024-11-20

### Fixed
- Critical security vulnerability
```

### Automated with Git

```bash
# Generate changelog from git log
git log v1.4.2..v1.5.0 --pretty=format:"- %s" --reverse

# With conventional commits
git log v1.4.2..v1.5.0 --pretty=format:"%s" | \
  grep "^feat:" | sed 's/^feat: /- /'
```

## GitHub Release Workflow

```bash
# 1. Create tag
git tag -a v1.5.0 -m "Release 1.5.0"
git push origin v1.5.0

# 2. Create release on GitHub
gh release create v1.5.0 \
  --title "Release 1.5.0" \
  --notes "New features and bug fixes" \
  dist/[app.zip](http://app.zip)

# 3. Or via GitHub web interface:
# - Go to Releases
# - Click "Draft a new release"
# - Select tag v1.5.0
# - Add release notes
# - Upload binaries
# - Publish
```

## Pre-release Workflow

```bash
# Beta release
git tag -a v2.0.0-beta.1 -m "Beta 1"
git push origin v2.0.0-beta.1

# Mark as pre-release on GitHub
gh release create v2.0.0-beta.1 --prerelease

# Release candidate
git tag -a v2.0.0-rc.1 -m "Release Candidate 1"
git push origin v2.0.0-rc.1

# Final release
git tag -a v2.0.0 -m "Major Release 2.0"
git push origin v2.0.0
```

## Hotfix Workflow

```bash
# Bug found in production (v1.5.0)

# 1. Branch from release tag
git checkout -b hotfix/1.5.1 v1.5.0

# 2. Fix bug
git commit -am "fix: critical security issue"

# 3. Tag hotfix
git tag -a v1.5.1 -m "Hotfix: Security patch"

# 4. Push
git push origin hotfix/1.5.1
git push origin v1.5.1

# 5. Merge back to main
git checkout main
git merge hotfix/1.5.1
git push origin main

# 6. Delete hotfix branch
git branch -d hotfix/1.5.1
git push origin --delete hotfix/1.5.1
```

## Multi-Version Support

```bash
# Support v1.x and v2.x simultaneously

# Main branches
main              # v2.x development
release/1.x       # v1.x maintenance

# Release v2.1.0
git checkout main
git tag v2.1.0
git push origin v2.1.0

# Backport critical fix to v1.x
git checkout release/1.x
git cherry-pick <commit-hash>
git tag v1.9.5
git push origin release/1.x
git push origin v1.9.5
```

## Release Checklist

### Before Release

- [ ]  All tests passing
- [ ]  Code reviewed
- [ ]  Documentation updated
- [ ]  CHANGELOG updated
- [ ]  Version bumped in code
- [ ]  Dependencies updated
- [ ]  Security scan completed

### During Release

- [ ]  Create tag with proper message
- [ ]  Push tag to remote
- [ ]  Create GitHub release
- [ ]  Upload build artifacts
- [ ]  Update release notes

### After Release

- [ ]  Verify deployment
- [ ]  Announce release
- [ ]  Monitor for issues
- [ ]  Update documentation site
- [ ]  Close milestone
- [ ]  Archive related branches

## Automated Release Pipeline

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build
        run: npm run build
      
      - name: Test
        run: npm test
      
      - name: Create Release
        uses: actions/create-release@v1
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
```

## Release Strategies

### Strategy 1: Continuous Deployment

```bash
# Every merge to main is a release
# Automated version bumping
# Good for: Web apps, SaaS

main → auto-test → auto-version → auto-deploy
```

### Strategy 2: Scheduled Releases

```bash
# Release every 2 weeks
# Batch features together
# Good for: Enterprise software

develop → release-branch → tag → deploy
```

### Strategy 3: Feature-Based Releases

```bash
# Release when feature complete
# Variable timeline
# Good for: Libraries, frameworks

feature-complete → test → tag → publish
```

## Common Release Patterns

### Pattern 1: Git Flow

```bash
main      # Production releases only
develop   # Integration branch
feature/* # Feature development
release/* # Release preparation
hotfix/*  # Production fixes
```

### Pattern 2: GitHub Flow

```bash
main       # Always deployable
feature/*  # Feature branches
# Simple, fast, good for CD
```

### Pattern 3: Trunk-Based

```bash
main  # Everyone commits here
# Short-lived branches (< 1 day)
# Feature flags for incomplete work
```

## Release Communication

### Release Notes Template

```markdown
# Release v1.5.0

## 🎉 New Features
- User export to CSV
- Bulk operations API
- Dark mode support

## 🐛 Bug Fixes
- Fixed login timeout
- Resolved memory leak
- Corrected date formatting

## ⚡ Performance
- 40% faster data loading
- Reduced memory usage

## 🔒 Security
- Patched XSS vulnerability
- Updated dependencies

## 📚 Documentation
- Updated API docs
- Added migration guide

## ⚠️ Breaking Changes
- Removed deprecated `/old-api` endpoint
- Changed config file format

## 🔄 Migration Guide
See [MIGRATION.md](http://MIGRATION.md) for details
```