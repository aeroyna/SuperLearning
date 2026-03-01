# Chapter 4: Branching & Merging

This is where git becomes **incredibly powerful** compared to Perforce. Branching in git is lightweight, fast, and central to the workflow.

## The Branching Mindset

**In Perforce:** Branching is heavy - you create depot paths, copy files, it's a big deal. You might have `//depot/main` and `//depot/release-1.0`

**In git:** Branches are just **pointers to commits**. Creating a branch is instant and costs nothing. You'll create branches all the time.

## Understanding Branches

```bash
# See all branches
git branch

# You'll see:
# * main  (the * shows which branch you're on)

# See where branches point
git log --oneline --all --graph
```

A branch is literally just a label pointing to a commit. That's it!

## Chapter Sections

[4.1 Creating and Switching Branches](Chapter%204%20Branching%20&%20Merging/4%201%20Creating%20and%20Switching%20Branches%202b96ba4e52e481f49b18f713000ca2f8.md)

[4.2 Understanding Branch Behavior](Chapter%204%20Branching%20&%20Merging/4%202%20Understanding%20Branch%20Behavior%202b96ba4e52e4815da844c46b5e1bc2a1.md)

[4.3 Merging Branches](Chapter%204%20Branching%20&%20Merging/4%203%20Merging%20Branches%202b96ba4e52e48135b261c23da6161744.md)

[4.4 Merge Conflicts](Chapter%204%20Branching%20&%20Merging/4%204%20Merge%20Conflicts%202b96ba4e52e481069658ee62892aa8ca.md)

[4.5 Branch Management](Chapter%204%20Branching%20&%20Merging/4%205%20Branch%20Management%202b96ba4e52e48153856fdb2cbedbb656.md)