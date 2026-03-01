# HotSpot Architecture

Oracle's HotSpot is the reference JVM implementation.

## 1. Key Components
*   **Class Loader:** Subsystem for loading types.
*   **Runtime Data Areas:** Heap, Metaspace, Stacks.
*   **Execution Engine:** Interpreter + JIT (C1/C2) + GC.
*   **Native Interface:** JNI for C++ calls.

## 2. Object Layout (OOPs)
In HotSpot, an object in the Heap has a header.
*   **Mark Word (8 bytes):** Hash code, GC age bits, Lock bits (for synchronized).
*   **Klass Pointer (4/8 bytes):** Points to the Class metadata in Metaspace.
*   **Instance Data:** Fields.
*   **Padding:** Objects are aligned to 8-byte boundaries.

## 3. Compressed Oops
On 64-bit systems, pointers are 8 bytes. This wastes cache.
*   **Optimization:** If Heap < 32GB, HotSpot uses 4-byte pointers ("Compressed Oops") representing *offsets* rather than addresses.
*   **Impact:** Running with a 31GB heap is often faster than a 35GB heap because 35GB forces 8-byte pointers, reducing effective cache size.