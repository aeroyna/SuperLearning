# Execution Contexts & The Call Stack 🧠

Understanding **Execution Context** is key to understanding Hoisting, Scope, and Closures in JavaScript.

---

## 1. What is an Execution Context?

An environment where JavaScript code is evaluated and executed. There are two types:

1.  **Global Execution Context (GEC)**: The default context. Creates the `window` object (in browsers) and `this`.
2.  **Function Execution Context (FEC)**: Created whenever a function is **invoked**.

---

## 2. The Creation Phase (Hoisting) 🏗️

Before code execution, the JS engine scans the code to create the context.

1.  **Creation of the Variable Object (VO)**:
    *   **Function Declarations**: Fully stored in memory. You can call them before definition.
    *   **`var` variables**: Stored as `undefined`.
    *   **`let` / `const`**: Stored in uninitialized state (Temporal Dead Zone). Accessing them throws ReferenceError.
2.  **Creation of Scope Chain**: Links to parent environments.
3.  **Determination of `this`**: Value is assigned.

### Example
```javascript
console.log(myVar); // undefined (Hoisted)
var myVar = 10;

console.log(myFunc()); // "Hello" (Hoisted)
function myFunc() { return "Hello"; }
```

---

## 3. The Execution Phase 🏃

The engine runs through the code line by line, assigning values and executing functions.

---

## 4. The Scope Chain & Closures 🔗

### Scope Chain
If a variable is not found in the current function's scope (VO), the engine looks at the **Lexical Parent** (where the function was physically written), continuing up to the Global Scope.

### Closures
A closure is a function bundled together with references to its surrounding state (the lexical environment).
*   Even if the outer function has finished executing (popped off the stack), the inner function **remembers** the variables from the outer scope because it maintains a reference to that memory.

```javascript
function makeCounter() {
  let count = 0; // "Closed over" variable
  return function() {
    count++;
    return count;
  };
}

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```
`makeCounter` finished running, but `count` persists in the heap because the inner function holds a reference to it.
