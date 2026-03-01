# Reference Types: Weak, Soft, Phantom

Java provides interaction with the Garbage Collector through special reference types in `java.lang.ref`. These allow you to hold references to objects without preventing them from being collected.

## 1. Strong Reference (The Default)
```java
Dog myDog = new Dog();
```
*   As long as `myDog` exists (is reachable from GC roots), the `Dog` object will **never** be collected, even if the JVM runs out of memory.

## 2. Soft Reference (Memory Sensitive)
```java
SoftReference<Dog> softDog = new SoftReference<>(new Dog());
```
*   **Behavior:** The GC guarantees to collect Softly reachable objects **before throwing an OutOfMemoryError**.
*   **Use Case:** **Memory-sensitive Caches**. If memory is plentiful, keep the cache. If memory is tight, dump the cache to survive.

## 3. Weak Reference (Canonical Mappings)
```java
WeakReference<Dog> weakDog = new WeakReference<>(new Dog());
```
*   **Behavior:** If an object is *only* reachable via Weak References, the GC will collect it **immediately** (in the next cycle). It does not wait for OOM.
*   **Use Case:** `WeakHashMap`, metadata association. "I want to reference this object as long as someone else is using it, but I don't want to keep it alive myself."

## 4. Phantom Reference (Post-Mortem Cleanup)
```java
ReferenceQueue<Dog> queue = new ReferenceQueue<>();
PhantomReference<Dog> phantomDog = new PhantomReference<>(new Dog(), queue);
```
*   **Behavior:** You cannot retrieve the object from a Phantom Reference (`get()` always returns null). It simply tracks when the object has been finalized/collected.
*   **Use Case:** Scheduling clean-up actions (like closing native buffers) *after* the object is dead, as a safer alternative to `finalize()`.

## 5. Reachability Hierarchy
1.  **Strongly Reachable:** Normal variables.
2.  **Softly Reachable:** Only via `SoftReference`.
3.  **Weakly Reachable:** Only via `WeakReference`.
4.  **Phantom Reachable:** Finalized, waiting for cleanup.
5.  **Unreachable:** Eligible for reclamation.