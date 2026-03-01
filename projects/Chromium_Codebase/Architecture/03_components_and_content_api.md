# Components & The Content API

**[⬅️ Back to Architecture Overview](00_architecture.md)**

---

## 🧩 The Core Separation: Chrome vs. Content

Chromium is not just one monolithic application. It is layered to separate the "Browser Engine" from the "Browser Application".

### 1. The `content/` Module
*   **What it is**: The core code needed to render a page using a multi-process browser.
*   **What it includes**:
    *   The Multi-process System (Browser/Renderer/GPU processes).
    *   IPC (Mojo).
    *   Sandboxing.
    *   Web Platform features (HTML5 implementation).
*   **What it EXCLUDES**:
    *   Tabs, Bookmarks, History.
    *   Extensions.
    *   Spellcheck, Autofill.
    *   The "Chrome" UI.
*   **Analogy**: If you were building a completely different browser (like Electron or a VR browser), you would build it on top of `content/`.

### 2. The `chrome/` Module
*   **What it is**: The "Google Chrome" (or Chromium) application.
*   **What it includes**:
    *   The UI (Omnibox, Toolbar, Settings pages).
    *   Features that glue `content` to the user (Sync, Sign-in).
    *   Extension system logic.

---

## 🔌 The Content API

How does `chrome/` talk to `content/` if `content/` doesn't know about `chrome/`? Through the **Content API**.

### 1. The Public Interface
Located in `content/public/`. This is the **only** part of `content` that embedders (like Chrome) are allowed to include.

*   **`content/public/browser/`**: Interfaces for the Browser process.
*   **`content/public/renderer/`**: Interfaces for the Renderer process.

### 2. Client Interfaces
`content` defines "Client" interfaces that `chrome` implements to handle specific logic.

*   **`ContentBrowserClient`**: "Hey Embedder, I'm about to start. Do you want to add any flags?"
*   **`WebContentsDelegate`**: "Hey Embedder, a web page wants to open a new window. How should I handle this?" (Chrome might open a new tab).

**Example Flow:**
1.  A renderer wants to open a popup.
2.  It sends an IPC to the Browser Process (`content`).
3.  `content` receives it but doesn't know about "Tabs".
4.  `content` calls `delegate_->AddNewContents()`.
5.  `chrome` (which implemented the delegate) receives the call and creates a new Tab Strip item.

---

## 👁️ Blink (`third_party/blink`)

Blink is the rendering engine. It started as a fork of WebKit.

*   **Location**: `third_party/blink/renderer/`.
*   **Role**: Implements the Web Standards (W3C).
*   **Core**:
    *   **Core**: DOM, HTML, CSS, SVG.
    *   **Modules**: Web Audio, IndexedDB, WebBluetooth (features that aren't core DOM).
    *   **Platform**: Low-level abstractions (threads, network wrappers).
*   **Language**: Heavy use of WTF (Web Template Framework), Blink's own version of STL strings/vectors optimized for performance.

---

## 🧱 Components (`components/`)

Code that is shared between `chrome/` and other embedders (like iOS Chrome, or Android WebView) lives in `components/`.

*   **Example**: `components/autofill`.
    *   Autofill logic is needed by Chrome on Desktop, Chrome on Android, and Chrome on iOS (which doesn't use `content/`).
    *   So the logic lives here, independent of `content/`.
