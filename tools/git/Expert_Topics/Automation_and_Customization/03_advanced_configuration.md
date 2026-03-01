# 12.3 Advanced Configuration

## Configuration Levels

```bash
# System-wide (all users)
git config --system [user.name](http://user.name) "Name"

# Global (current user)
git config --global [user.name](http://user.name) "Name"

# Local (current repo)
git config --local [user.name](http://user.name) "Name"

# Check where config comes from
git config --list --show-origin

# Priority: local > global > system
```

## Essential Configuration

```bash
# Identity
git config --global [user.name](http://user.name) "Your Name"
git config --global [user.email](http://user.email) "[your.email@example.com](mailto:your.email@example.com)"

# Editor
git config --global core.editor "vim"
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "nano"         # Nano

# Default branch name
git config --global init.defaultBranch main

# Line endings (Windows)
git config --global core.autocrlf true

# Line endings (Mac/Linux)
git config --global core.autocrlf input
```

## Diff and Merge Tools

```bash
# Use VS Code for diff
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Use VS Code for merge
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Use vimdiff
git config --global merge.tool vimdiff
git config --global diff.tool vimdiff

# Launch diff tool
git difftool

# Launch merge tool
git mergetool
```

## Performance Settings

```bash
# Enable parallel index preload
git config --global core.preloadindex true

# Enable file system cache (Windows)
git config --global core.fscache true

# Increase http post buffer (for large pushes)
git config --global http.postBuffer 524288000

# Enable commit graph (faster operations)
git config --global core.commitGraph true
git config --global gc.writeCommitGraph true

# Auto garbage collection
git config --global [gc.auto](http://gc.auto) 256
```

## Color Configuration

```bash
# Enable colors
git config --global color.ui auto

# Disable colors
git config --global color.ui false

# Custom colors for branches
git config --global color.branch.current "yellow reverse"
git config --global color.branch.local "yellow"
git config --global color.branch.remote "green"

# Custom colors for status
git config --global color.status.added "green"
git config --global color.status.changed "yellow"
git config --global color.status.untracked "red"

# Custom colors for diff
git config --global color.diff.meta "blue"
git config --global color.diff.frag "magenta bold"
git config --global color.diff.old "red"
git config --global [color.diff.new](http://color.diff.new) "green"
```

## Push and Pull Behavior

```bash
# Default push behavior
git config --global push.default simple    # Push current branch
git config --global push.default current   # Always push to branch with same name
git config --global push.default upstream  # Push to upstream branch

# Auto-setup remote tracking
git config --global push.autoSetupRemote true

# Pull strategy
git config --global pull.rebase true   # Rebase instead of merge on pull
git config --global pull.ff only       # Only fast-forward merges

# Show push status
git config --global push.default simple
```

## Credential Storage

```bash
# Cache credentials for 1 hour
git config --global credential.helper cache
git config --global credential.helper 'cache --timeout=3600'

# Store credentials permanently (less secure)
git config --global credential.helper store

# Use OS keychain (Mac)
git config --global credential.helper osxkeychain

# Use Windows Credential Manager
git config --global credential.helper wincred

# Use Git Credential Manager
git config --global credential.helper manager
```

## Submodule Configuration

```bash
# Auto-update submodules
git config --global submodule.recurse true

# Show submodule summary
git config --global status.submoduleSummary true

# Show submodule diff
git config --global diff.submodule log
```

## Merge Configuration

```bash
# Merge strategy
git config --global merge.ff false        # Always create merge commit
git config --global merge.ff only         # Only fast-forward

# Show conflict style
git config --global merge.conflictStyle diff3  # Show base, ours, theirs

# Auto-resolve conflicts
git config --global merge.renameLimit 999999
```

## Rebase Configuration

```bash
# Auto-stash before rebase
git config --global rebase.autoStash true

# Auto-squash fixup commits
git config --global rebase.autoSquash true

# Update refs
git config --global rebase.updateRefs true
```

## Commit Configuration

```bash
# Use verbose commits (show diff)
git config --global commit.verbose true

# Sign commits with GPG
git config --global commit.gpgSign true
git config --global user.signingkey YOUR_KEY_ID

# Commit template
git config --global commit.template ~/.gitmessage
```

## Viewing Configuration

```bash
# List all config
git config --list

# List with origins
git config --list --show-origin

# Show specific value
git config [user.name](http://user.name)

# Show all user settings
git config --get-regexp user

# Edit config file directly
git config --global --edit
```

## Removing Configuration

```bash
# Unset specific value
git config --global --unset [user.name](http://user.name)

# Remove entire section
git config --global --remove-section alias
```

## Conditional Configuration

```bash
# Different configs for different directories
vim ~/.gitconfig

[user]
    name = Personal Name
    email = [personal@email.com](mailto:personal@email.com)

[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work

[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal

# ~/.gitconfig-work
[user]
    name = Work Name
    email = [work@company.com](mailto:work@company.com)
```

## Git Attributes

```bash
# Create .gitattributes in repo root
vim .gitattributes

# Treat as binary (no diff)
*.png binary
*.jpg binary
*.pdf binary

# Always use LF line endings
*.sh text eol=lf

# Always use CRLF line endings
*.bat text eol=crlf

# Auto-detect text files
* text=auto

# Custom diff for specific files
*.json diff=json

# Filter for secrets
secrets.txt filter=secret
```

## Ignore Files

```bash
# Global gitignore
vim ~/.gitignore_global

# OS files
.DS_Store
Thumbs.db

# Editor files
.vscode/
.idea/
*.swp

# Configure Git to use it
git config --global core.excludesfile ~/.gitignore_global
```

## Best Configuration Practices

1. **Set identity first** - name and email
2. **Use global for personal preferences** - editor, colors
3. **Use local for project-specific** - email for work projects
4. **Document custom configs** - add comments
5. **Backup your config** - version control ~/.gitconfig
6. **Use conditional includes** - work vs personal
7. **Review periodically** - clean up unused settings

## Recommended Global Config

```bash
[user]
    name = Your Name
    email = [your.email@example.com](mailto:your.email@example.com)

[core]
    editor = code --wait
    autocrlf = input
    excludesfile = ~/.gitignore_global
    
[init]
    defaultBranch = main
    
[push]
    default = current
    autoSetupRemote = true
    
[pull]
    rebase = true
    
[rebase]
    autoStash = true
    autoSquash = true
    
[merge]
    conflictStyle = diff3
    
[color]
    ui = auto
    
[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    lg = log --graph --oneline --decorate --all
```