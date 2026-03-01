# Threads and Processes in React Native

React Native applications are not single-threaded in the traditional browser sense. They rely on a multi-threaded architecture to maintain 60fps (frames per second) while executing business logic.

## The Three Main Threads

### 1. The Main Thread (UI Thread)
This is the standard native thread allocated by iOS (Main Queue) or Android (UI Thread).
*   **Responsibilities:**
    *   Handling user interactions (touches, gestures).
    *   Rendering native UI components.
    *   Calculating layout (if not offloaded).
*   **Constraint:** Blocking this thread drops frames, causing the app to "freeze" or stutter.

### 2. The JavaScript Thread
This is where your React application logic lives.
*   **Responsibilities:**
    *   Executing JavaScript code (React diffing, state updates, business logic).
    *   Processing API calls.
    *   Sending instructions to the Native side (e.g., "Create a View").
*   **Constraint:** If this thread is busy (e.g., heavy calculation), the UI might remain responsive to scrolling (on the Native thread), but updates (like pressing a button to change a number) will lag.

### 3. The Shadow Thread (Layout Thread)
This is a background thread used by React Native.
*   **Responsibilities:**
    *   Calculating layout using the Yoga engine (Flexbox implementation).
    *   Constructing the "Shadow Tree" (a representation of the UI layout).
*   **Process:** The JS thread sends component information here -> Yoga calculates size/position -> Information is sent to the Main Thread for rendering.

## Interaction Flow
1.  **Touch Event:** User taps a button. The **Main Thread** captures this.
2.  **Message Passing:** The event is serialized and sent to the **JS Thread**.
3.  **Processing:** React processes the `onClick`. State changes.
4.  **Update:** React schedules an update.
5.  **Layout:** The **Shadow Thread** recalculates the layout if necessary.
6.  **Render:** Instructions are sent back to the **Main Thread** to update the native view.

## Common Pitfalls
*   **Blocking JS:** Doing heavy computation (image processing, large list filtering) on the JS thread delays UI updates.
*   **Bridge Congestion:** Sending too much data (serialized JSON) back and forth between threads can cause bottlenecks (in the legacy architecture).
