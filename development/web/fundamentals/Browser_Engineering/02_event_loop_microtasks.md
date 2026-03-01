# The JavaScript Event Loop & Microtasks 🔄

JavaScript is **single-threaded**, meaning it can only execute one task at a time. The **Event Loop** is the mechanism that allows JS to perform non-blocking I/O operations (like network requests or timers) despite being single-threaded.

---

## 1. Architecture Overview 🏗️

### The Call Stack (LIFO)
*   Where the JS engine keeps track of what function is currently running.
*   If you call a function, it's pushed onto the stack. When it returns, it's popped off.

### Web APIs (The Browser)
*   Functionality provided by the browser (not the JS engine itself).
*   Includes `setTimeout`, `fetch`, `DOM Events`.
*   When you call `setTimeout`, the browser handles the timer, not the main JS thread.

### The Callback Queue (Task Queue)
*   Holds callbacks from Web APIs (e.g., the function inside `setTimeout`).
*   **Priority**: Low.

### The Microtask Queue
*   Holds promises (`.then/catch/finally`) and `MutationObserver` callbacks.
*   **Priority**: High.

---

## 2. How the Event Loop Works ⚙️

The algorithm is a continuous loop:

1.  **Execute Script**: Run synchronous code on the Call Stack until empty.
2.  **Process Microtasks**: Check the Microtask Queue. Run **ALL** microtasks until the queue is completely empty.
    *   *Note*: If a microtask schedules another microtask, it is also run immediately, potentially blocking the UI.
3.  **Render**: (Optional) If the browser needs to repaint, it does so now.
4.  **Process ONE Task**: Pick the oldest task from the Callback Queue (Macrotask) and push it to the Call Stack.
5.  **Repeat**.

---

## 3. Code Example 💻

```javascript
console.log('1. Script Start');

setTimeout(() => {
  console.log('2. setTimeout');
}, 0);

Promise.resolve().then(() => {
  console.log('3. Promise 1');
}).then(() => {
  console.log('4. Promise 2');
});

console.log('5. Script End');
```

### Execution Order:
1.  `'1. Script Start'` (Sync)
2.  `setTimeout` is sent to Web API.
3.  `Promise` is sent to Microtask Queue.
4.  `'5. Script End'` (Sync)
5.  **Stack Empty. Check Microtasks.**
6.  `'3. Promise 1'` (Microtask)
7.  `'4. Promise 2'` (Microtask - chained)
8.  **Microtasks Empty. Check Macrotasks.**
9.  `'2. setTimeout'` (Macrotask)

**Output**: 1 -> 5 -> 3 -> 4 -> 2.
