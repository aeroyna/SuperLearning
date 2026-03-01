# Chapter 5: Remote Repositories

Now we're getting to the "server" part! This is where git connects to GitHub, GitLab, or any remote repository.

## The Distributed Model

**In Perforce:** One central server. You `p4 sync` to get latest, `p4 submit` to send changes.

**In git:** Multiple repositories can exist. Your local repo is a full copy. You `push` your commits to remotes and `pull` commits from remotes.

```
Your laptop          GitHub/GitLab          Teammate's laptop
   (repo)      <-->     (repo)        <-->      (repo)
```

All three are **complete repositories** with full history!

## Chapter Sections

[5.1 Understanding Remotes](Chapter%205%20Remote%20Repositories/5%201%20Understanding%20Remotes%202b96ba4e52e481f1b13fcaa2913aa155.md)

[5.2 Cloning Repositories](Chapter%205%20Remote%20Repositories/5%202%20Cloning%20Repositories%202b96ba4e52e481978813ef0f188c8fd3.md)

[5.3 Pushing Changes](Chapter%205%20Remote%20Repositories/5%203%20Pushing%20Changes%202b96ba4e52e481209920df96df181672.md)

[5.4 Pulling Changes](Chapter%205%20Remote%20Repositories/5%204%20Pulling%20Changes%202b96ba4e52e481a69880c5d36ac7cd0e.md)

[5.5 Working with Remote Branches](Chapter%205%20Remote%20Repositories/5%205%20Working%20with%20Remote%20Branches%202b96ba4e52e48152b798d5743178cb04.md)

[5.6 Common Workflows](Chapter%205%20Remote%20Repositories/5%206%20Common%20Workflows%202b96ba4e52e481948ae3e0c8b9112b6d.md)