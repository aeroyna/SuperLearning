# Contribution Workflow

**[⬅️ Back to Contribution Guide](00_contribution_guide.md)**

---

## 🔄 The Lifecycle of a Patch

Chromium does not use GitHub Pull Requests. It uses **Gerrit** for code review and **git-cl** (part of depot_tools) to manage submissions.

### 1. Create a Branch
Always start a new branch for your feature or fix.

```bash
git checkout -b my-awesome-feature origin/main
```

### 2. Make Changes
Edit files, compile, and test.
```bash
# Verify your changes
autoninja -C out/Default chrome
./out/Default/chrome
```

### 3. Commit Locally
Follow the [commit message guidelines](https://chromium.googlesource.com/chromium/src/+/main/docs/contributing.md#cl-description).

```bash
git commit -a
```
*   **Format**:
    *   Line 1: Summary (Under 72 chars).
    *   Line 2: Blank.
    *   Line 3+: Detailed description.
    *   Last Line: `Bug: 12345` (Link to Monorail/Issue Tracker).

### 4. Upload to Gerrit
This creates a CL (Change List) on the review server.

```bash
git cl upload
```
*   This will prompt you to edit the description if needed.
*   It runs presubmit checks (linting).
*   It gives you a link: `https://chromium-review.googlesource.com/c/chromium/src/+/123456`

### 5. Request Review
1.  Open the Gerrit link.
2.  Click **Start Review**.
3.  Add reviewers.
    *   **Finding Reviewers**: Look at the `OWNERS` file in the directory you modified. You need approval from an OWNER.
    *   **Tip**: `git cl owners` can suggest reviewers.

### 6. Iterate
*   Reviewers will leave comments.
*   Fix code locally.
*   **Amend** your commit (do not create new commits).
    ```bash
    git commit -a --amend
    ```
*   Upload the new patchset.
    ```bash
    git cl upload
    ```

### 7. Landing (Commit Queue)
Once you have `LGTM` (Looks Good To Me) from all owners:
1.  Click **Submit to CQ** (Commit Queue) in Gerrit.
2.  The CQ runs all tests on bots.
3.  If green, it automatically merges into `main`.

---

## 🕵️ Finding Something to Work On

1.  **Good First Issues**: Search the issue tracker for `Hotlist=GoodFirstBug`.
2.  **Code Health**: Look for `CodeHealth` rotation tasks (refactoring, cleanup).
3.  **Typos/Comments**: Start small by fixing documentation.
