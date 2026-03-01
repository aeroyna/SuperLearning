# Stack and Tab Patterns

Mobile navigation relies on two primary primitives: the Stack and the Tabs.

## 1. The Stack Navigator
*   **Model:** LIFO (Last In, First Out).
*   **Action:** You "push" a new screen on top. You "pop" (go back) to remove it.
*   **Memory:** As you push deeper, previous screens remain mounted in memory (though `react-native-screens` helps optimize this).
*   **UX:** Standard for drill-down flows (List -> Item Details -> Edit Item).

## 2. The Tab Navigator
*   **Model:** Parallel active history.
*   **Action:** Switching tabs does not destroy the state of the previous tab.
*   **Memory:** All tabs are generally mounted simultaneously (lazy loading is an option).
*   **UX:** Top-level application sections (Feed, Search, Profile).

## Nesting Navigators
A common pattern is a **Stack inside a Tab**, or a **Tab inside a Stack**.

### Example: "Instagram" Model
1.  **Root:** Tab Navigator (Feed, Explore, Profile).
2.  **Feed Tab:** A Stack Navigator.
    *   Index: Feed List.
    *   Push: Post Details.
    *   Push: User Profile (from a comment).

**Pitfall:** Navigating "across" stacks. If you are in the Feed Stack and click a link to "Edit Profile" (which lives in the Profile Tab's stack), the router must handle switching the parent Tab *and* pushing to the child Stack.
