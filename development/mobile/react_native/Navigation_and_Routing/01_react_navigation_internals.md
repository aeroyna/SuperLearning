# React Navigation Internals

The standard library for routing in the community is **React Navigation**.

## JS-Based vs. Native-Based

### Historically (JS-Based)
Early versions of React Navigation implemented transitions (slide animations) entirely in JavaScript using `Animated`.
*   **Pros:** wildly customizable, identical behavior across platforms.
*   **Cons:** Performance. If the JS thread was blocked, the navigation transition would stutter or freeze.

### Modern (Native-Backed)
Modern React Navigation (v5/v6+) integrates with `react-native-screens`.
*   **Concept:** Instead of just absolute positioning JS Views, it uses the native OS navigation fragments.
    *   **iOS:** `UINavigationController`
    *   **Android:** `Fragment` / `Activity`
*   **Benefit:** The OS manages the transition and the memory state of screens "behind" the current one. The OS can free up memory from background screens more effectively.

## State Management
React Navigation keeps the entire app's navigation state in a massive Javascript Object Context.
*   **Serialization:** This state can be serialized to JSON, saved to disk, and restored on app relaunch to bring the user back exactly where they were.
*   **Structure:**
    ```json
    {
      "index": 1,
      "routes": [
        { "name": "Home" },
        { "name": "Profile", "params": { "userId": 42 } }
      ]
    }
    ```
