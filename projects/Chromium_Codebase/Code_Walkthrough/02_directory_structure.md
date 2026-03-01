# Directory Structure Map

**[⬅️ Back to Code Walkthrough Overview](00_code_walkthrough.md)**

---

## 🗺️ The Source Tree

The root `src/` directory is vast. Here are the neighborhoods you need to know.

### 🏢 The Big Three

*   **`chrome/`**: The Application.
    *   `browser/`: The Browser Process code for Chrome features (UI, Sync, Extensions).
    *   `renderer/`: Chrome-specific code running in the Renderer Process.
    *   `test/`: Integration tests for Chrome.
*   **`content/`**: The Engine.
    *   `public/`: The API exposed to embedders (like Chrome).
    *   `browser/`: Internal implementation of the Browser Process.
    *   `renderer/`: Internal implementation of the Renderer Process.
*   **`third_party/`**: External Dependencies.
    *   `blink/`: The Rendering Engine.
    *   `skia/`: The 2D Graphics Library.
    *   `v8/`: The JavaScript Engine.

### 🛠️ Infrastructure

*   **`base/`**: Common Utilities.
    *   "Chrome's Standard Library". Strings, Threading, Files, Time, memory management.
    *   **Rule**: Everyone can depend on `base`. `base` depends on no one.
*   **`net/`**: Networking Stack.
    *   DNS, TCP/IP, HTTP/2/3, QUIC.
*   **`ui/`**: UI Frameworks.
    *   `views/`: The desktop UI toolkit (Windows, Linux, ChromeOS).
    *   `gfx/`: Graphics primitives (Points, Rects, Colors).
    *   `events/`: Input events (Keyboard, Mouse).

### 🧩 Shared Code

*   **`components/`**: Modular features shared between embedders.
    *   `autofill/`, `omnibox/`, `password_manager/`.
    *   Separated to be used by Chrome on iOS (which uses WebKit) vs Chrome on Android/Desktop.

### 🤖 Build & Tools

*   **`build/`**: Build configuration (GN templates).
*   **`tools/`**: Scripts and developer tools.
