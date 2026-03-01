# Chromium Glossary

**[⬅️ Back to Reference](00_reference.md)**

---

## 🗣️ Common Terminology

You will see these acronyms everywhere in chats, bugs, and code reviews.

*   **CL (Change List)**: A set of changes (commits) submitted for review. Equivalent to a Pull Request (PR) in GitHub.
*   **CQ (Commit Queue)**: The automated system that runs tests and merges your CL if tests pass.
*   **LGTM (Looks Good To Me)**: Approval from a reviewer. You usually need at least one LGTM from an OWNER.
*   **TBR (To Be Reviewed)**: *Deprecated*. Used to mean "submit now, review later".
*   **OWNERS**: Files that list who is responsible for a directory. You need approval from someone in this file to land code there.
*   **LKGR (Last Known Good Revision)**: The latest commit that passed all tests.
*   **ToT (Tip of Tree)**: The very latest commit on the main branch (might be broken).
*   **Sheriff**: A rotating role for a developer who watches the build bots and reverts CLs that break the tree.
*   **Gardener**: Similar to Sheriff, often specific to a sub-team (e.g., Blink Gardener).
*   **Monorail**: The issue tracker (bugs.chromium.org). Note: Being migrated to Google Issue Tracker.
*   **Trybot**: A build bot that runs tests on your CL *before* it lands.
*   **Waterfall**: The UI showing the status of all build bots.

---

## 🏗️ Architecture Terms

*   **Browser Process**: The main process.
*   **Renderer**: The process that renders the page.
*   **Utility Process**: A sandbox process for untrusted parsing or services.
*   **Mojo**: The IPC system.
*   **BrowserContext**: Essentially a "Profile" (Incognito has its own BrowserContext).
*   **WebContents**: A tab.
*   **RFH (RenderFrameHost)**: The browser-side object for a frame.
