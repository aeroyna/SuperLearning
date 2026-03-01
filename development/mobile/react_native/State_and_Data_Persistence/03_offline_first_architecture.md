# Offline First Architecture

"Offline First" means the app works primarily from the local cache and synchronizes with the network in the background.

## Core Concepts

### 1. The Cache is the Source of Truth
The UI always renders from the local database (MMKV/SQLite/WatermelonDB).
*   **Read Path:** UI <- Local DB.
*   **Write Path:** UI -> Local DB -> Network (Sync).

### 2. Optimistic Updates
When the user clicks "Like":
1.  **Immediately** update the local UI to show the heart.
2.  **Queue** the API request.
3.  **Rollback** if the API request fails (and show an error toast).
*   *Why?* Makes the app feel instant, regardless of network latency.

### 3. Synchronization Queue
If the device is offline, mutations (POST/PUT/DELETE) must be stored in a persistent queue.
1.  User creates a Post (offline).
2.  App saves "Pending Post" to local DB.
3.  Network monitor detects `online`.
4.  App processes the queue: sends the POST request.
5.  On success, replaces "Pending Post" with the real confirmed data.

## Challenges
*   **Conflict Resolution:** What if the user edits the same item on two devices while offline? (Last Write Wins vs. Merge Strategies).
*   **IDs:** Generating temporary IDs for offline items that are later replaced by server DB IDs.
