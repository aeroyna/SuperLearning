# Server State vs. UI State

A common anti-pattern in early React Native apps was storing *everything* in a global Redux store: API responses, form inputs, navigation state, and UI toggles.

## The Problem
*   **Boilerplate:** Writing thunks/sagas for every API call.
*   **Staleness:** Manually tracking `isLoading`, `isError`, and invalidating cache is hard.
*   **Performance:** A massive single state tree can cause unnecessary re-renders if not memoized correctly.

## The Distinction

### 1. Server State
Data that persists remotely and we only have a *snapshot* of.
*   *Examples:* User profile, List of tweets, Search results.
*   *Characteristics:* Asynchronous, shared by other users, can be out of date.
*   *Solution:* **TanStack Query (React Query)** or **SWR**. These libraries handle caching, polling, deduplication, and "stale-while-revalidate" logic automatically.

### 2. Client (UI) State
Data that exists only in the app session.
*   *Examples:* Is the modal open? What is typed in the search bar? Which theme is selected?
*   *Solution:*
    *   **Local:** `useState`, `useReducer` (for single components).
    *   **Global:** `Zustand`, `Jotai`, or `Context API` (for lightweight global settings).

## Modern Approach
Use React Query for the 90% of data that comes from the API. Use Zustand/Context for the remaining 10% of global UI settings.
