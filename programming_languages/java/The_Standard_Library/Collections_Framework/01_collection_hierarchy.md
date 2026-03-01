# Collection Hierarchy

The Java Collections Framework (JCF) is a unified architecture for representing and manipulating collections (groups of objects). It reduces programming effort, increases performance, and fosters software reuse.

## 1. The Big Picture

The framework is built upon two main root interfaces:
1.  **`java.util.Collection`**: Represents a group of objects known as elements.
2.  **`java.util.Map`**: Represents mappings between keys and values. *Note: `Map` does not extend `Collection`.*

```
        Collection                      Map
       /    |     \                      |
   List    Set    Queue              SortedMap
    |       |       |
ArrayList HashSet PriorityQueue      TreeMap
LinkedList TreeSet   Deque
```

## 2. The `Collection` Interface

This is the root interface for the collection hierarchy. It defines the basic operations that all collections (Lists, Sets, Queues) share.

### Common Methods
*   `add(E e)`: Adds an element.
*   `remove(Object o)`: Removes an element.
*   `contains(Object o)`: Checks if an element exists.
*   `size()`: Returns the number of elements.
*   `isEmpty()`: Checks if the collection is empty.
*   `iterator()`: Returns an iterator over the elements.
*   `clear()`: Removes all elements.
*   `toArray()`: Returns an array containing all elements.

## 3. Sub-Interfaces of `Collection`

### 3.1 `List` (Ordered Collection)
*   **Characteristics:** An ordered sequence of elements.
*   **Duplicates:** Allowed.
*   **Access:** Positional access via integer index (`get(int index)`).
*   **Implementations:** `ArrayList`, `LinkedList`, `Vector` (legacy).

### 3.2 `Set` (Unique Elements)
*   **Characteristics:** A collection that contains no duplicate elements. Models the mathematical set abstraction.
*   **Duplicates:** Not allowed.
*   **Access:** No positional access (generally).
*   **Implementations:** `HashSet`, `LinkedHashSet`, `TreeSet`.

### 3.3 `Queue` (Holding for Processing)
*   **Characteristics:** Designed for holding elements prior to processing. Typically First-In-First-Out (FIFO).
*   **Implementations:** `PriorityQueue`, `LinkedList` (implements `Deque`).

## 4. The `Map` Interface

Maps are not Collections in the strict inheritance sense, but they are fully integrated into the framework.

*   **Characteristics:** An object that maps **keys** to **values**.
*   **Constraints:** A map cannot contain duplicate keys; each key can map to at most one value.
*   **Implementations:** `HashMap`, `LinkedHashMap`, `TreeMap`, `Hashtable` (legacy).

### Common Map Methods
*   `put(K key, V value)`: Associates value with key.
*   `get(Object key)`: Returns value associated with key.
*   `remove(Object key)`: Removes the mapping for a key.
*   `containsKey(Object key)`: Checks if a key exists.
*   `keySet()`: Returns a `Set` view of the keys.
*   `values()`: Returns a `Collection` view of the values.
*   `entrySet()`: Returns a `Set` view of the mappings (`Map.Entry`).

## 5. `Collections` Utility Class

Do not confuse the **interface** `Collection` with the **utility class** `Collections`.
*   `java.util.Collections`: Contains static methods that operate on or return collections (e.g., `sort()`, `shuffle()`, `reverse()`, `unmodifiableList()`).

```java
List<String> names = new ArrayList<>();
names.add("Bob");
names.add("Alice");

// Use the utility class to sort
Collections.sort(names); 
```

## 6. Iterator Interface

The `Iterator` interface provides a standard way to traverse through a collection one element at a time.

```java
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    // Safe removal during iteration
    if (name.equals("Bob")) {
        it.remove();
    }
}
```
*   **Enhanced For-Loop:** Internally uses the Iterator.

Understanding this hierarchy helps you choose the right data structure for your specific needs (e.g., "Do I need unique items?" -> `Set`. "Do I need key-value pairs?" -> `Map`).
