# Blink Rendering Engine

**[⬅️ Back to Architecture Overview](00_architecture.md)**

---

## 🎨 The Rendering Pipeline

Blink is the engine that transforms HTML/CSS/JS into pixels. This process is a complex pipeline known as the **Rendering Lifecycle**.

### 1. Parsing & DOM Construction
*   **Input**: HTML byte stream.
*   **Output**: Document Object Model (DOM) Tree.
*   **Classes**: `HTMLDocumentParser`, `HTMLTreeBuilder`, `Document`.
*   **Note**: This happens incrementally as bytes arrive from the network.

### 2. Style Recalculation (Style)
*   **Input**: DOM Tree + CSS stylesheets.
*   **Output**: Computed Style for every element.
*   **Classes**: `StyleResolver`, `ComputedStyle`.
*   **Process**: Matches CSS selectors to DOM nodes and resolves "cascade" logic (precedence).

### 3. Layout (LayoutNG)
*   **Input**: DOM Tree + Computed Style.
*   **Output**: Layout Tree (Fragment Tree).
*   **Goal**: Determine geometry (x, y, width, height) and line breaks.
*   **LayoutNG (Next Generation)**: The modern layout engine.
    *   **Constraint Spaces**: Parents pass down constraints (e.g., "You have 200px width").
    *   **Immutable Fragments**: Layout produces immutable fragments, making it easier to cache and parallelize.
    *   **Classes**: `LayoutObject`, `NGBlockNode`, `NGConstraintSpace`.

### 4. Paint
*   **Input**: Layout Tree.
*   **Output**: Paint Artifacts (Display Items).
*   **Goal**: Record *what* to draw, not *how* to draw it.
*   **Process**:
    *   Walks the layout tree.
    *   Generates a list of instructions: "Draw background blue", "Draw text 'Hello'".
    *   **Classes**: `PaintController`, `DisplayItemList`.

### 5. Compositing
*   **Input**: Paint Artifacts.
*   **Output**: Compositor Layers (Tiles).
*   **Goal**: Separate the page into independent layers that can be scrolled or transformed cheaply on the GPU.
*   **Property Trees**: Blink builds specific trees for Transform, Clip, Effect, and Scroll nodes to avoid re-rasterizing the whole page on scroll.
*   **Classes**: `PaintArtifactCompositor`, `cc::Layer`.

---

## 🏗️ Blink Architecture

### 1. The "Renderer" Directory
Located at `third_party/blink/renderer/`.
*   **`core/`**: DOM, HTML, CSS, Layout, Paint. The heart of the engine.
*   **`modules/`**: Features that sit on top of core (WebAudio, IndexedDB, WebXR).
*   **`platform/`**: Low-level abstractions (graphics, network, scheduling) that abstract away the underlying OS/Chrome details.
*   **`bindings/`**: The glue between C++ (Blink) and V8 (JavaScript).

### 2. Oilpan (Blink GC)
Blink uses its own Garbage Collector called **Oilpan**.
*   **Why?**: To handle cyclic references between DOM objects and JS wrappers.
*   **Usage**:
    *   Inherit from `GarbageCollected<T>`.
    *   Use `Member<T>` instead of raw pointers.
    *   Implement `Trace(Visitor*)` to tell the GC about your pointers.
