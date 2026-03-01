# Stack vs Heap: A Deep Dive

Understanding the distinction between Stack and Heap is fundamental to understanding Java's memory model.

## 1. The Stack: Fast and Focused

### Structure
The Stack is a LIFO (Last-In-First-Out) data structure. It is contiguous in memory, which makes access extremely fast (CPU cache friendly).

### What goes on the Stack?
*   **Primitives:** `int x = 10;`. The value `10` is stored directly on the stack.
*   **References:** `Dog myDog = ...`. The variable `myDog` (which is just a memory address, like `0x1234`) is stored on the stack.

### Thread Safety
The Stack is **Thread-Private**. No other thread can access your stack. Therefore, local variables are inherently thread-safe.

## 2. The Heap: The Object Ocean

### Structure
The Heap is a large, fragmented pool of memory. Allocating memory here involves finding a free block of sufficient size.

### What goes on the Heap?
*   **Objects:** `new Dog()`. The actual `Dog` object (with its fields `name`, `breed`) is allocated here.
*   **Instance Variables:** Fields of an object (even primitives) live inside the object on the Heap.
    *   *Example:* If `Dog` has `int age`, the value of `age` is stored inside the `Dog` object on the Heap, not on the stack.

### Thread Safety
The Heap is **Shared**. Any thread with a reference to an object can access/modify it. This requires synchronization to prevent race conditions.

## 3. Example Execution

```java
public void method1() {
    int i = 1;           // 1. Stack: i = 1
    Object obj = new Object(); // 2. Stack: obj (ref) -> Heap: Object
    method2(obj);
}

public void method2(Object param) { // 3. Stack (Frame 2): param (copy of ref) -> Same Heap Object
    String s = "Hello";  // 4. Stack: s (ref) -> Heap: String "Hello" (String Pool)
}
```

1.  **Frame 1 (method1):** Created. `i` and `obj` reference stored here.
2.  **Heap:** `new Object()` allocated.
3.  **Frame 2 (method2):** Pushed on top. `param` receives a copy of the reference from `obj`. Both point to the same Heap object.
4.  **Exit:** `method2` finishes. Frame 2 popped. `s` and `param` disappear.
5.  **GC:** The "Hello" string might be collected if nothing else references it (though String Pool rules apply).