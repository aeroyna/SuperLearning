# Flexbox Layout Engine (Yoga)

React Native does not use a browser layout engine (like Gecko or Blink). It uses **Yoga**, a standalone layout engine written in C++.

## What is Yoga?
Yoga implements a subset of the CSS Flexbox specification. It is designed to run on the CPU and calculate coordinates (top, left, width, height) for every element in the tree.

## Key Differences from Web Flexbox

### 1. Default Direction
*   **Web:** `flex-direction: row` (Items flow left-to-right).
*   **React Native:** `flex-direction: column` (Items flow top-to-bottom).
    *   *Why?* Mobile screens are portrait-oriented; vertical lists are the standard.

### 2. Missing Properties
Yoga is strictly Flexbox. It does not support:
*   CSS Grid.
*   `float`, `block`, `inline-block`.
*   Relative units like `em`, `rem`, `vh`, `vw` (though RN provides Dimension APIs to simulate these).

### 3. Absolute Positioning
In React Native, `position: 'absolute'` works relative to the **nearest parent**, regardless of whether that parent has `position: 'relative'`.
*   *Note:* This behavior simplifies the model but can confuse web developers.

## Performance Implications
Yoga runs on the Shadow Thread.
1.  **Dirty Nodes:** When you change a style (e.g., `width`), Yoga marks the node as "dirty".
2.  **Propagation:** It may need to recalculate the parent (if the parent shrank) or children (if they were constrained).
3.  **Optimization:** Avoid layout thrashing (rapidly changing layout properties in animation frames). Use `transform` (GPU handled) instead of `top/left` (CPU layout handled) for animations.
