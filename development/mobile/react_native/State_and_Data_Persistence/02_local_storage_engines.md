# Local Storage Engines

Persisting data on the device is essential for keeping the user logged in and storing preferences.

## 1. AsyncStorage
The built-in (now extracted community) solution.
*   **Architecture:** Asynchronous, key-value store.
*   **Implementation:**
    *   **Android:** SQLite under the hood.
    *   **iOS:** Serialized dictionary files.
*   **Pros:** Simple API, works everywhere.
*   **Cons:** **Slow**. Every read/write goes across the Bridge. Large JSON blobs block the JS thread during parsing.

## 2. MMKV
The modern standard for key-value storage.
*   **Architecture:** Direct bindings to Tencent's MMKV (C++) using JSI.
*   **Pros:** **Synchronous** and blazing fast (100x faster than AsyncStorage). Uses memory mapping (mmap) to keep data synced with the file system without heavy I/O cost.
*   **Use Case:** User settings, auth tokens, small to medium JSON objects.

## 3. SQLite (via `react-native-quick-sqlite` or similar)
Full relational database.
*   **Architecture:** C++ bindings to SQLite.
*   **Pros:** Complex queries, relations, ACID compliance.
*   **Use Case:** Storing thousands of items (e.g., an offline generic catalog) where you need to filter/sort efficiently without loading everything into JS memory.

## 4. Realm / WatermelonDB
Object-oriented databases that support lazy loading.
*   **Concept:** You don't load the whole array; you query for objects, and fields are accessed lazily.
*   **Use Case:** extremely complex offline-first apps with heavy data relationships.
