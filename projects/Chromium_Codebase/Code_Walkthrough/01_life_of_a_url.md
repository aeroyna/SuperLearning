# Life of a URL: Code Walkthrough

**[⬅️ Back to Code Walkthrough Overview](00_code_walkthrough.md)**

---

## 🚦 From Omnibox to Pixel

Understanding the path a URL takes from the moment you hit "Enter" to the moment the page renders is the "Hello World" of understanding Chromium internals.

### Phase 1: The Browser Process (UI & Network)

1.  **User Input**:
    *   The user types into the Omnibox.
    *   **Code**: `LocationBarView` (Views) or `OmniboxView` handles input.
    *   **Autocomplete**: `AutocompleteController` fetches suggestions.

2.  **Navigation Start**:
    *   User hits Enter.
    *   **Code**: `NavigationController::LoadURL`.
    *   This creates a `NavigationEntry` and notifies `WebContents`.

3.  **Network Request**:
    *   The browser checks if a renderer process already exists for this site (Process Model).
    *   **Code**: `NavigationURLLoader` sends a request to the **Network Service**.
    *   The Network Service (via `URLLoader`) performs DNS resolution, TLS handshake, and sends the HTTP GET request.

4.  **Response Handling**:
    *   The Network Service receives the response headers.
    *   It checks for downloads or redirects.
    *   If it's HTML, the browser process prepares a renderer.
    *   **Code**: `RenderFrameHostImpl::CommitNavigation`.

### Phase 2: The Renderer Process (Blink)

1.  **Commit**:
    *   The Browser sends a "CommitNavigation" Mojo message to the Renderer.
    *   The Renderer initializes a `DocumentLoader`.

2.  **Parsing (HTML)**:
    *   The stream of data arrives.
    *   **Code**: `HTMLDocumentParser` tokenizes HTML string into nodes.
    *   **DOM Tree**: Nodes are constructed into the DOM (Document Object Model) tree.
    *   **Preload Scanner**: Scans ahead for `<img>` and `<link>` tags to start fetching resources early.

3.  **Style Calculation (CSS)**:
    *   CSS is parsed into `CSSStyleSheet`.
    *   **Code**: `StyleResolver` matches CSS rules to DOM elements.
    *   **Computed Style**: Every element gets a final set of style properties (color, font-size).

4.  **Layout (Reflow)**:
    *   Calculates geometry (x, y, width, height) of every element.
    *   **Code**: `LayoutObject` tree is built (separate from DOM tree).
    *   Layout engine (LayoutNG) computes constraint spaces and fragment trees.

5.  **Paint**:
    *   Records "draw" instructions (draw text here, draw rect there).
    *   **Code**: `PaintLayer` and `DisplayItemList`.
    *   This produces a list of painting operations, not pixels yet.

6.  **Compositing**:
    *   Splits the page into layers (scrolling, z-index, transforms).
    *   **Code**: `Compositor` (cc module).
    *   Layers are turned into **Tiles**.

### Phase 3: The GPU Process (Raster & Display)

1.  **Rasterization**:
    *   The Compositor sends tiles to the GPU Process.
    *   **Skia**: The graphics library (similar to Cairo or HTML5 Canvas) executes the paint operations to generate bitmaps (pixels) for each tile.
    *   This is often hardware-accelerated (GL/Vulkan).

2.  **Draw Quad**:
    *   The Renderer sends "Draw Quads" (references to rasterized tiles) to the Browser/GPU compositor (Viz).

3.  **Display**:
    *   **Viz (Visuals)**: The system service responsible for displaying frames.
    *   It aggregates quads from the browser UI and the web page renderer.
    *   It performs the final "Swap Buffers" to put pixels on the screen.

---

## 🗝️ Key Classes to Search

Use `git grep` or [source.chromium.org](https://source.chromium.org) to find these:

*   **`content::NavigationController`**: Manages the back/forward list.
*   **`content::WebContents`**: The "Tab" object. Prime for plugging in features.
*   **`content::RenderFrameHost`**: The browser-side representation of a frame (main frame or iframe).
*   **`blink::LocalFrame`**: The renderer-side representation of a frame.
*   **`blink::LayoutObject`**: Base class for all layout nodes.
