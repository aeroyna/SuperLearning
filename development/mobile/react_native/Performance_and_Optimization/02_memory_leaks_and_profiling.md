# Memory Leaks and Profiling

A memory leak occurs when the app holds onto RAM that it no longer needs, eventually causing the OS to kill the app.

## Common Leaks in React Native

### 1. Unmounted Component Updates
*   **Scenario:** You start an API call in `useEffect`. The user navigates away (component unmounts). The API returns, and you call `setState`.
*   **Fix:** Clean up subscriptions/flags in the `useEffect` cleanup function.

### 2. Event Listeners
*   **Scenario:** `DeviceEventEmitter.addListener(...)` without removing it.
*   **Result:** The listener closure holds references to the component scope, keeping it alive forever.

### 3. Timers
*   **Scenario:** `setInterval` running in the background.

## Profiling Tools

### Flipper (Deprecated/Optional)
Historically used, but now being phased out for native tools.

### Android Studio Profiler
1.  Open `android` folder in Android Studio.
2.  Run App.
3.  Click "Profiler" tab.
4.  **Memory:** Watch the graph. If it climbs endlessly as you navigate in and out of a screen, you have a leak. Dump the Java Heap to find retained instances.

### Xcode Instruments (iOS)
1.  Open `ios` workspace in Xcode.
2.  Product -> Profile -> **Leaks**.
3.  Navigate your app. The tool will flag leaked objects and retain cycles.
