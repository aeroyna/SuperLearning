# Iterator and Iterable

The `Iterator` pattern provides a standard way to traverse a collection of elements one by one, regardless of the underlying implementation (Array, Linked List, Tree).

## 1. The `Iterable` Interface

*   **Contract:** Any class implementing `Iterable<T>` must provide a method `iterator()` that returns an `Iterator<T>`.
*   **Significance:** Only objects that implement `Iterable` can be used in the Java **Enhanced For-Loop** (for-each).
*   All `Collection` interfaces (`List`, `Set`, etc.) extend `Iterable`. `Map` does not.

## 2. The `Iterator` Interface

Defines how to move through the collection.

### Methods
1.  `boolean hasNext()`: Returns `true` if there are more elements.
2.  `E next()`: Returns the next element. Throws `NoSuchElementException` if no more elements.
3.  `void remove()`: Removes the last element returned by the iterator. This is the **only safe way** to remove elements during iteration.

### Usage
```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");

// Explicit Iterator
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    System.out.println(s);
}

// Enhanced For-Loop (Implicit Iterator)
for (String s : list) {
    System.out.println(s);
}
```

## 3. Fail-Fast Iterators

Most collection implementations in `java.util` (ArrayList, HashMap, etc.) return **Fail-Fast** iterators.

*   **Mechanism:** They keep a modification count (`modCount`).
*   **Behavior:** If the collection is structurally modified (added/removed elements) by any means *except* through the iterator's own `remove()` method while iterating, the iterator throws a **`ConcurrentModificationException`**.

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");

for (String s : list) {
    if (s.equals("A")) {
        // list.remove(s); // THROWS ConcurrentModificationException!
    }
}
```

## 4. `ListIterator` (List Only)
An extension of `Iterator` specifically for Lists.
*   **Bidirectional:** Can go `previous()` and `next()`.
*   **Modification:** Can `set()` and `add()` elements during iteration.
