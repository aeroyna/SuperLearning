# State and Data Persistence

Data management on mobile has unique challenges: flaky networks, limited battery, and the need to persist data across app kills.

## In this chapter, you will learn:

*   **[4.1 Server State vs UI State](01_server_state_vs_ui_state.md)**
    *   Why Redux is often misused.
    *   Using React Query / TanStack Query for server data.

*   **[4.2 Local Storage Engines](02_local_storage_engines.md)**
    *   `AsyncStorage` (The old standard) vs. MMKV (The fast standard) vs. SQLite (The relational standard).
    *   Performance benchmarks and serialization costs.

*   **[4.3 Offline First Architecture](03_offline_first_architecture.md)**
    *   Optimistic UI updates.
    *   Queueing mutations for later synchronization.
