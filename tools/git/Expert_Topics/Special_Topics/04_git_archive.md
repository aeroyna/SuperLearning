# 13.4 Git Archive

## What is Git Archive?

Git archive creates tar/zip files of your repository without .git directory.

**Use cases:**

- Create release packages
- Share code without Git history
- Deploy to servers
- Create backups
- Distribute source code

## Basic Archive Usage

```bash
# Create zip of current branch
git archive --format=zip --output=[project.zip](http://project.zip) HEAD

# Create tar.gz
git archive --format=tar.gz --output=project.tar.gz HEAD

# Archive specific branch
git archive --format=zip --output=[release.zip](http://release.zip) v1.0.0

# Archive specific commit
git archive --format=zip --output=[snapshot.zip](http://snapshot.zip) abc1234
```

## Archive Specific Paths

```bash
# Archive specific directory
git archive --format=zip --output=[src.zip](http://src.zip) HEAD src/

# Archive multiple paths
git archive --format=zip --output=[subset.zip](http://subset.zip) HEAD src/ docs/ [README.md](http://README.md)

# Archive with pattern
git archive --format=zip --output=[js-only.zip](http://js-only.zip) HEAD "*.js"
```

## Archive with Prefix

```bash
# Add directory prefix inside archive
git archive --format=zip --prefix=my-project-1.0/ --output=[release.zip](http://release.zip) v1.0.0

# When extracted:
# my-project-1.0/
#   ├── src/
#   ├── [README.md](http://README.md)
#   └── ...

# Useful for distribution
git archive --format=tar.gz --prefix=myproject/ HEAD | gzip > release.tar.gz
```

## Excluding Files from Archive

```bash
# Use .gitattributes to exclude files
echo "tests/ export-ignore" >> .gitattributes
echo "*.test.js export-ignore" >> .gitattributes
echo ".github/ export-ignore" >> .gitattributes
echo "secrets/ export-ignore" >> .gitattributes

git add .gitattributes
git commit -m "Configure archive exclusions"

# Now archive excludes these
git archive --format=zip --output=[clean.zip](http://clean.zip) HEAD
```

## Archive to stdout

```bash
# Pipe to gzip
git archive HEAD | gzip > project.tar.gz

# Pipe over SSH
git archive HEAD | ssh user@server "cd /deploy && tar -xf -"

# Create and extract in one command
git archive HEAD | tar -x -C /tmp/extracted/
```

## Automation Examples

### Create Timestamped Release

```bash
# Create timestamped release
DATE=$(date +%Y%m%d)
VERSION=$(git describe --tags --always)
git archive --format=tar.gz --prefix=project-${VERSION}/ --output=project-${DATE}.tar.gz HEAD
```

### Create Release for Each Tag

```bash
# Create release archives for all tags
mkdir -p releases
for tag in $(git tag); do
    git archive --format=zip --prefix=$tag/ --output=releases/$[tag.zip](http://tag.zip) $tag
    echo "Created releases/$[tag.zip](http://tag.zip)"
done
```

### Deploy Script

```bash
#!/bin/bash
# [deploy.sh](http://deploy.sh) - Deploy to server

SERVER="[deploy@production.com](mailto:deploy@production.com)"
DEPLOY_DIR="/var/www/app"

# Create archive and deploy
git archive HEAD | ssh $SERVER "
    cd $DEPLOY_DIR && 
    tar -xf - && 
    echo 'Deployed successfully'
"
```

## Archive Best Practices

1. **Use prefix** - Makes extraction cleaner
2. **Exclude unnecessary files** - Use export-ignore
3. **Include version** - In filename and prefix
4. **Document** - Include README in archive
5. **Test extraction** - Verify archive contents
6. **Sign releases** - Use GPG for important releases
7. **Automate** - Use scripts or CI/CD

## Archive vs Clone

| Feature | Archive | Clone |
| --- | --- | --- |
| Size | Smaller | Larger |
| History | No | Yes |
| .git | No | Yes |
| Speed | Faster | Slower |
| Use case | Distribution | Development |