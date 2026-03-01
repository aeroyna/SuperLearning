# Chapter 1: The Git Mindset

## Understanding the Paradigm Shift

**Perforce thinking:** Centralized server, check out files, submit changelists

**Git thinking:** Distributed, everyone has full history, commit locally then push

## Key Mind Shift

- **In Perforce:** you work against THE server
- **In Git:** you work in YOUR local repository, sync with remotes when ready
- Commits are local and cheap - you commit often, push when done

## The Three Trees

Git operates on three distinct layers:

1. **Working Directory** - your actual files (like Perforce workspace)
2. **Staging Area (Index)** - what will go in next commit (Perforce doesn't have this!)
3. **Repository (.git folder)** - all your history (like Perforce depot, but local)

## Your First Commands

```bash
# Check git version
git --version

# Set your identity (do this once)
git config --global [user.name](http://user.name) "Your Name"
git config --global [user.email](http://user.email) "[you@email.com](mailto:you@email.com)"

# See your config
git config --list
```

## Quick Exercise

```bash
# Create a practice folder
mkdir git-practice
cd git-practice

# Initialize a git repo (like p4 client setup, but creates the "server" locally)
git init

# Check status (you'll use this constantly)
git status
```

You should see "On branch main" and "No commits yet".

## Key Concept to Internalize

Git is optimistic. In Perforce you "check out" to signal intent. In git, you just edit files freely, then decide what to commit.