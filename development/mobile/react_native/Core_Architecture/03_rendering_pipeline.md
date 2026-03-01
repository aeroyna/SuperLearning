# Rendering Pipeline: React vs. Native

Understanding how a React component becomes pixels on the screen involves tracing the data flow from JSX to Native Views.

## The Three Trees

1.  **React Element Tree:** The output of your React components (pure JS objects).
2.  **Shadow Tree:** A C++ tree representation used by the Yoga layout engine to calculate geometry.
3.  **Host View Tree:** The actual Native UI hierarchy (Android `View`s, iOS `UIView`s).

## The Pipeline Process

### 1. Render Phase (JavaScript)
*   React executes your component logic (`render()`, hooks).
*   It generates the **React Element Tree**.
*   It calculates the "diff" between the previous tree and the new tree.

### 2. Commit Phase (Bridge/JSI)
*   **Legacy:** Operations are batched and sent via the Bridge as JSON instructions ("Create View ID 4", "Update Opacity ID 2").
*   **Fabric (New Architecture):** The Render phase in JS can directly create the Shadow Tree nodes in C++ via JSI. This is the "Render Pipeline" in Fabric.

### 3. Layout Phase (Shadow Thread)
*   The **Shadow Tree** contains style information (Flexbox).
*   **Yoga** (the layout engine) traverses this tree to calculate absolute X, Y, Width, and Height for every node.

### 4. Mounting Phase (Main Thread)
*   The layout data and view properties are applied to the **Host View Tree**.
*   Native views are created, updated, or destroyed.
*   The OS renders the pixels.

## Fabric: The Concurrent Renderer
The New Architecture (Fabric) brings React 18's concurrency to Native.
*   **Prioritization:** React can interrupt a low-priority render (background data fetch) to handle a high-priority update (user input).
*   **Synchronous Layout:** React can measure text and layout synchronously, eliminating "layout jumps" common in the asynchronous bridge era.
