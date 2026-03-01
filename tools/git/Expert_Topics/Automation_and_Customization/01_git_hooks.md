# 12.1 Git Hooks

## What are Git Hooks?

Git hooks are scripts that run automatically at specific points in Git's workflow.

**Use cases:**

- Enforce commit message format
- Run tests before push
- Check code style before commit
- Notify team on push
- Auto-deploy on push
- Prevent commits to specific branches

## Hook Locations

```bash
# Hooks are in .git/hooks/
ls .git/hooks/

# Sample hooks (remove .sample to activate)
applypatch-msg.sample
commit-msg.sample
pre-commit.sample
pre-push.sample
prepare-commit-msg.sample
# ... and more
```

## Types of Hooks

**Client-side hooks (local):**

- `pre-commit` - Before commit is created
- `prepare-commit-msg` - Before commit message editor opens
- `commit-msg` - After commit message is entered
- `post-commit` - After commit is completed
- `pre-rebase` - Before rebase
- `post-checkout` - After checkout
- `post-merge` - After merge
- `pre-push` - Before push

**Server-side hooks (on remote):**

- `pre-receive` - Before refs are updated
- `update` - Once per ref being updated
- `post-receive` - After refs are updated

## Creating a Simple Hook

```bash
# Create pre-commit hook
vim .git/hooks/pre-commit

#!/bin/bash
echo "Running pre-commit checks..."

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed! Commit aborted."
    exit 1
fi

echo "All checks passed!"
exit 0

# Make executable
chmod +x .git/hooks/pre-commit
```

## Pre-Commit Hook Examples

### Example 1: Check for debug code

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check for console.log
if git diff --cached | grep -E "console\.(log|debug|info)"; then
    echo "Error: console.log found in staged files"
    exit 1
fi

exit 0
```

### Example 2: Run linter

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Get staged JS files
FILES=$(git diff --cached --name-only --diff-filter=ACMR | grep '\.js$')

if [ -n "$FILES" ]; then
    echo "Running ESLint..."
    npx eslint $FILES
    if [ $? -ne 0 ]; then
        echo "ESLint failed! Fix errors before committing."
        exit 1
    fi
fi

exit 0
```

### Example 3: Check file size

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Reject files larger than 5MB
MAX_SIZE=5242880  # 5MB in bytes

for file in $(git diff --cached --name-only); do
    if [ -f "$file" ]; then
        size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
        if [ $size -gt $MAX_SIZE ]; then
            echo "Error: $file is larger than 5MB ($size bytes)"
            exit 1
        fi
    fi
done

exit 0
```

## Commit-Msg Hook Examples

### Example 1: Enforce conventional commits

```bash
#!/bin/bash
# .git/hooks/commit-msg

COMMIT_MSG=$(cat "$1")
PATTERN="^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+"

if ! echo "$COMMIT_MSG" | grep -Eq "$PATTERN"; then
    echo "Error: Commit message doesn't follow conventional format"
    echo "Format: <type>(<scope>): <subject>"
    echo "Example: feat(auth): add login functionality"
    exit 1
fi

exit 0
```

### Example 2: Add issue number

```bash
#!/bin/bash
# .git/hooks/prepare-commit-msg

BRANCH=$(git rev-parse --abbrev-ref HEAD)
ISSUE=$(echo $BRANCH | grep -oE '[A-Z]+-[0-9]+')

if [ -n "$ISSUE" ]; then
    # Add issue number to commit message
    echo "$ISSUE: $(cat $1)" > $1
fi
```

## Pre-Push Hook Examples

### Example 1: Run tests before push

```bash
#!/bin/bash
# .git/hooks/pre-push

echo "Running tests before push..."
npm test

if [ $? -ne 0 ]; then
    echo "Tests failed! Push aborted."
    exit 1
fi

exit 0
```

### Example 2: Prevent push to main

```bash
#!/bin/bash
# .git/hooks/pre-push

BRANCH=$(git rev-parse --abbrev-ref HEAD)

if [ "$BRANCH" = "main" ]; then
    echo "Error: Direct push to main is not allowed!"
    echo "Please create a pull request instead."
    exit 1
fi

exit 0
```

## Post-Commit Hook Examples

### Example 1: Notify team

```bash
#!/bin/bash
# .git/hooks/post-commit

# Send notification to Slack
curl -X POST -H 'Content-type: application/json' \
    --data "{\"text\":\"New commit by $(git config [user.name](http://user.name)): $(git log -1 --pretty=%B)\"}" \
    [https://hooks.slack.com/services/YOUR/WEBHOOK/URL](https://hooks.slack.com/services/YOUR/WEBHOOK/URL)
```

### Example 2: Update ticket

```bash
#!/bin/bash
# .git/hooks/post-commit

COMMIT_MSG=$(git log -1 --pretty=%B)
ISSUE=$(echo $COMMIT_MSG | grep -oE 'JIRA-[0-9]+')

if [ -n "$ISSUE" ]; then
    # Update JIRA ticket
    curl -X POST "[https://your-jira.com/rest/api/2/issue/$ISSUE/comment](https://your-jira.com/rest/api/2/issue/$ISSUE/comment)" \
        -H "Content-Type: application/json" \
        -d "{\"body\": \"Commit: $(git rev-parse --short HEAD)\"}"
fi
```

## Bypassing Hooks

```bash
# Skip pre-commit and commit-msg hooks
git commit --no-verify -m "Emergency fix"

# Skip pre-push hook
git push --no-verify
```

<aside>
⚠️

**Warning:** Only bypass hooks in emergencies. They exist to protect code quality!

</aside>

## Sharing Hooks with Team

**Problem:** `.git/hooks/` is not tracked by Git.

### Solution 1: Store in repo

```bash
# Create hooks directory in repo
mkdir .githooks
mv .git/hooks/pre-commit .githooks/

# Configure Git to use this directory
git config core.hooksPath .githooks

# Team members run same command after clone
```

### Solution 2: Setup script

```bash
# [setup-hooks.sh](http://setup-hooks.sh)
#!/bin/bash
cp hooks/* .git/hooks/
chmod +x .git/hooks/*
echo "Hooks installed!"

# Team runs after clone:
# ./[setup-hooks.sh](http://setup-hooks.sh)
```

### Solution 3: Use Husky (Node.js)

```bash
npm install husky --save-dev
npx husky init

# Creates .husky/ directory with hooks
# Automatically installs on npm install
```

## Hook Best Practices

1. **Keep hooks fast** - Developers will bypass slow hooks
2. **Provide clear error messages** - Explain what went wrong
3. **Make hooks optional** - Allow bypass for emergencies
4. **Share with team** - Use core.hooksPath or Husky
5. **Test hooks** - They're code too!
6. **Document hooks** - Explain what each hook does
7. **Use exit codes** - 0 = success, non-zero = failure

## Advanced Hook: Preventing Force Push

```bash
#!/bin/bash
# .git/hooks/pre-push

while read local_ref local_sha remote_ref remote_sha
do
    if [ "$local_sha" = "0000000000000000000000000000000000000000" ]; then
        continue
    fi
    
    if [ "$remote_sha" = "0000000000000000000000000000000000000000" ]; then
        continue
    fi
    
    # Check if this is a force push
    if ! git merge-base --is-ancestor "$remote_sha" "$local_sha"; then
        BRANCH=$(echo "$remote_ref" | sed 's/refs\/heads\///')
        if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "develop" ]; then
            echo "Error: Force push to $BRANCH is not allowed!"
            exit 1
        fi
    fi
done

exit 0
```

## Common Hook Patterns

### Pattern 1: Pre-commit quality checks

```bash
#!/bin/bash
echo "Running quality checks..."

# Lint
npm run lint || exit 1

# Format check
npm run format:check || exit 1

# Type check
npm run type-check || exit 1

echo "All checks passed!"
```

### Pattern 2: Commit message validation

```bash
#!/bin/bash
COMMIT_MSG=$(cat "$1")

# Check minimum length
if [ ${#COMMIT_MSG} -lt 10 ]; then
    echo "Error: Commit message too short (min 10 chars)"
    exit 1
fi

# Check for ticket reference
if ! echo "$COMMIT_MSG" | grep -qE '#[0-9]+'; then
    echo "Warning: No ticket reference found"
fi
```

### Pattern 3: Pre-push protection

```bash
#!/bin/bash
# Prevent direct push to protected branches

PROTECTED_BRANCHES="main develop"
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

for branch in $PROTECTED_BRANCHES; do
    if [ "$CURRENT_BRANCH" = "$branch" ]; then
        echo "Error: Direct push to $branch not allowed!"
        exit 1
    fi
done
```