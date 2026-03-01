# 12.2 Git Aliases

## What are Git Aliases?

Aliases are custom shortcuts for Git commands.

## Creating Aliases

```bash
# Basic aliases
git config --global [alias.co](http://alias.co) checkout
git config --global [alias.br](http://alias.br) branch
git config --global [alias.ci](http://alias.ci) commit
git config --global [alias.st](http://alias.st) status

# Now you can use:
git co main
git br
git ci -m "message"
git st
```

## Useful Aliases

```bash
# Pretty log
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# Undo last commit
git config --global alias.undo "reset HEAD~1 --mixed"

# Amend without editing message
git config --global alias.amend "commit --amend --no-edit"

# List aliases
git config --global alias.aliases "config --get-regexp ^alias\."

# Show branches sorted by date
git config --global alias.recent "branch --sort=-committerdate"

# Delete merged branches
git config --global alias.cleanup "!git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d"
```

## Complex Aliases with Shell Commands

```bash
# Aliases starting with ! are shell commands

# Create branch and switch
git config --global alias.cob '!git checkout -b'

# Add all and commit
git config --global [alias.ac](http://alias.ac) '!git add -A && git commit'

# Push and set upstream
git config --global alias.publish '!git push -u origin $(git branch --show-current)'

# Show files changed in last commit
git config --global alias.changed '!git diff --name-only HEAD~1'

# Interactive rebase last N commits
git config --global alias.reb '!f() { git rebase -i HEAD~$1; }; f'
# Usage: git reb 5
```

## Productivity Aliases

```bash
# Sync with main
git config --global alias.sync '!git fetch origin && git rebase origin/main'

# Quick save work
git config --global [alias.save](http://alias.save) '!git add -A && git commit -m "WIP"'

# Resume work
git config --global alias.resume 'reset HEAD~1'

# What did I do today?
git config --global [alias.today](http://alias.today) "log --since='midnight' --oneline --author=$(git config [user.email](http://user.email))"

# Show contributors
git config --global alias.contributors "shortlog -sn"

# Show current branch
git config --global alias.current "rev-parse --abbrev-ref HEAD"
```

## Aliases in ~/.gitconfig

```bash
# Edit gitconfig
vim ~/.gitconfig

[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    
    # Pretty log
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    
    # Undo
    undo = reset HEAD~1 --mixed
    
    # Amend
    amend = commit --amend --no-edit
    
    # List aliases
    aliases = config --get-regexp ^alias\.
    
    # Recent branches
    recent = branch --sort=-committerdate
```

## Best Aliases Collection

```bash
[alias]
    # Basic shortcuts
    co = checkout
    br = branch
    ci = commit
    st = status
    
    # Logging
    lg = log --graph --oneline --decorate --all
    ll = log --oneline --decorate --all
    last = log -1 HEAD --stat
    
    # Undoing
    undo = reset HEAD~1 --mixed
    unstage = reset HEAD --
    
    # Branching
    cob = checkout -b
    cleanup = !git branch --merged | grep -v '\\*\\|main' | xargs git branch -d
    
    # Committing
    ac = !git add -A && git commit
    save = !git add -A && git commit -m 'SAVEPOINT'
    wip = !git add -u && git commit -m "WIP"
    
    # Syncing
    sync = !git fetch origin && git rebase origin/main
    update = !git fetch origin && git merge origin/$(git branch --show-current)
    
    # Info
    contributors = shortlog -sn
    today = log --since=midnight --oneline --author=$(git config [user.email](http://user.email))
    week = log --since='1 week ago' --oneline --author=$(git config [user.email](http://user.email))
```

## Advanced Alias Examples

### Alias with parameters

```bash
# Search commits by message
git config --global alias.find '!f() { git log --all --grep="$1"; }; f'
# Usage: git find "bug fix"

# Create branch from issue number
git config --global alias.issue '!f() { git checkout -b "feature/issue-$1"; }; f'
# Usage: git issue 123

# Compare branches
git config --global [alias.compare](http://alias.compare) '!f() { git log $1..$2 --oneline; }; f'
# Usage: git compare main develop
```

### Workflow aliases

```bash
# Start new feature
git config --global alias.feature '!f() { git checkout main && git pull && git checkout -b "feature/$1"; }; f'

# Start hotfix
git config --global alias.hotfix '!f() { git checkout main && git pull && git checkout -b "hotfix/$1"; }; f'

# Finish feature (squash merge)
git config --global alias.finish '!f() { 
    BRANCH=$(git branch --show-current);
    git checkout main && 
    git pull && 
    git merge --squash $BRANCH && 
    git commit && 
    git branch -d $BRANCH;
}; f'
```

### Information aliases

```bash
# Show ignored files
git config --global alias.ignored "ls-files --others --ignored --exclude-standard"

# Show untracked files
git config --global alias.untracked "ls-files --others --exclude-standard"

# Show file history
git config --global alias.filelog "log -u"

# Show who worked on file
git config --global alias.who "shortlog -sn --"
```

## Viewing Aliases

```bash
# List all aliases
git aliases

# OR
git config --get-regexp alias

# Show specific alias
git config alias.lg
```

## Removing Aliases

```bash
# Remove alias
git config --global --unset [alias.co](http://alias.co)

# Remove all aliases (dangerous!)
git config --global --remove-section alias
```

## Alias Best Practices

1. **Keep names short but memorable** - `co` better than `c`
2. **Don't override built-in commands** - avoid `alias.log`
3. **Document complex aliases** - add comments in .gitconfig
4. **Share with team** - keep team aliases in sync
5. **Test before adding** - make sure they work
6. **Use functions for parameters** - `!f() { ... }; f`
7. **Group related aliases** - organize in .gitconfig

## Team Aliases

```bash
# Create team aliases file
vim ~/.gitconfig-team

[alias]
    # Team workflow aliases
    feature = !git checkout develop && git pull && git checkout -b
    
# Include in main config
vim ~/.gitconfig
[include]
    path = ~/.gitconfig-team
```

## Conditional Aliases

```bash
# Different aliases for work vs personal
vim ~/.gitconfig

[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work

[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```