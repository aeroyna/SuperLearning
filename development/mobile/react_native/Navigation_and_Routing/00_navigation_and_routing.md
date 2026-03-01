# Navigation and Routing

Navigation is often cited as the hardest part of mobile development. Unlike the web (where the URL drives state), mobile navigation requires managing a complex stack of screens, history, and transitions.

## In this chapter, you will learn:

*   **[3.1 React Navigation Internals](01_react_navigation_internals.md)**
    *   JS-based navigators vs. Native-backed navigators (`react-native-screens`).
    *   How the navigation state tree is managed.

*   **[3.2 Stack and Tab Patterns](02_stack_and_tab_patterns.md)**
    *   The "Card Stack" model (push/pop) vs. the "Tab" model (parallel history).
    *   Nesting navigators correctly.

*   **[3.3 Deep Linking and Intents](03_deep_linking_and_intents.md)**
    *   Mapping external URLs (`myapp://user/1`) to specific navigation states.
    *   Android Intents vs. iOS Universal Links.
