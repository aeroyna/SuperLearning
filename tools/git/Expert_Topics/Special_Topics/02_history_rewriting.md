# 13.2 History Rewriting

## When to Rewrite History

**Valid reasons:**

- Remove sensitive data (passwords, keys)
- Remove large files accidentally committed
- Clean up before making repo public
- Fix email/name in old commits

<aside>
🚨

**DANGER:** Rewriting published history breaks things for others. Only do this if absolutely necessary and coordinate with your entire team!

</aside>

## Using filter-repo (Recommended)

```bash
# Install git-filter-repo
pip install git-filter-repo

# Remove file from all history
git filter-repo --path sensitive.txt --invert-paths

# Remove directory from history
git filter-repo --path secrets/ --invert-paths

# Remove multiple paths
git filter-repo --path-glob '*.key' --invert-paths

# Replace text in all commits
git filter-repo --replace-text expressions.txt

# expressions.txt:
# password123==>***REMOVED***
# secret_key==>***REMOVED***
```

## Using BFG Repo Cleaner (Fast)

```bash
# Download from [https://rtyley.github.io/bfg-repo-cleaner/](https://rtyley.github.io/bfg-repo-cleaner/)

# Remove passwords
java -jar bfg.jar --replace-text passwords.txt repo.git

# Remove large files (bigger than 50MB)
java -jar bfg.jar --strip-blobs-bigger-than 50M repo.git

# Remove specific files
java -jar bfg.jar --delete-files secrets.txt repo.git

# Clean up after BFG
cd repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

## Remove Sensitive Data - Complete Process

```bash
# 1. Backup first!
cp -r repo repo-backup

# 2. Remove sensitive data
git filter-repo --path secrets.txt --invert-paths

# 3. Clean up
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 4. Verify it's gone
git log --all -- secrets.txt  # Should be empty

# 5. Force push (COORDINATE WITH TEAM!)
git push origin --force --all
git push origin --force --tags

# 6. Notify team to re-clone
```

## After Rewriting History - Team Actions

```bash
# Option 1: Hard reset (lose local changes)
git fetch origin
git reset --hard origin/main

# Option 2: Re-clone (safest)
cd ..
rm -rf old-repo
git clone [https://github.com/user/repo.git](https://github.com/user/repo.git)

# Option 3: Rebase local changes
git fetch origin
git rebase origin/main
```

## Preventing Sensitive Data

### Use .gitignore

```bash
# Add to .gitignore
echo "secrets.txt" >> .gitignore
echo ".env" >> .gitignore
echo "*.key" >> .gitignore
echo "*.pem" >> .gitignore

git add .gitignore
git commit -m "Ignore sensitive files"
```

### Use git-secrets

```bash
# Install git-secrets
brew install git-secrets

# Set up in repo
git secrets --install
git secrets --register-aws

# Add custom patterns
git secrets --add 'password\s*=\s*.+'
git secrets --add 'api[_-]?key\s*=\s*.+'

# Scan for secrets
git secrets --scan
git secrets --scan-history
```

## Changing Author in History

```bash
# Change author for all commits
git filter-branch --env-filter '
OLD_EMAIL="[old@email.com](mailto:old@email.com)"
NEW_NAME="New Name"
NEW_EMAIL="[new@email.com](mailto:new@email.com)"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$NEW_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$NEW_NAME"
    export GIT_AUTHOR_EMAIL="$NEW_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

## History Rewrite Best Practices

1. **Backup first** - Copy entire repo before rewriting
2. **Coordinate with team** - Everyone must re-clone or reset
3. **Test on copy** - Try rewrite on repo copy first
4. **Verify results** - Check history is clean
5. **Revoke secrets** - If credentials leaked, change them!
6. **Document process** - Write down steps for team