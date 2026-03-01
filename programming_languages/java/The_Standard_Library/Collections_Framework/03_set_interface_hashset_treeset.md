# Set Interface: HashSet and TreeSet

The `Set` interface models the mathematical set abstraction. Its defining characteristic is that **it cannot contain duplicate elements**.

## 1. `HashSet`

`HashSet` is the most commonly used implementation of `Set`.

### Internal Structure
*   **Backed by a HashMap:** Internally, a `HashSet` uses a `HashMap` to store its elements. The elements you add to the `Set` are stored as **keys** in the `HashMap`, and a dummy object is used as the value.
*   **Ordering:** **Unordered**. The order of elements is not guaranteed and can change over time.

### Requirements
*   Elements MUST implement `equals()` and `hashCode()` correctly. Uniqueness relies entirely on these methods.

### Characteristics
*   **Add/Remove/Contains:** **O(1)** (constant time) on average. This makes it extremely efficient for checking membership.
*   **Nulls:** Allows one `null` element.

```java
Set<String> fruits = new HashSet<>();
fruits.add("Apple");
fruits.add("Banana");
fruits.add("Apple"); // Duplicate ignored, returns false

System.out.println(fruits); // Output: [Apple, Banana] (Order not guaranteed)
```

## 2. `TreeSet`

`TreeSet` implements the `SortedSet` and `NavigableSet` interfaces.

### Internal Structure
*   **Red-Black Tree:** Internally uses a `TreeMap` (Red-Black tree).
*   **Ordering:** **Sorted**. Elements are stored in their natural ordering (e.g., A-Z, 1-10) or by a custom `Comparator` provided at creation.

### Characteristics
*   **Add/Remove/Contains:** **O(log n)**. Slower than `HashSet` but guaranteed logarithmic time.
*   **Nulls:** Does **NOT** allow `null` elements (throws `NullPointerException`).

### Use Case
Use `TreeSet` when you need unique elements **and** you need them to be sorted automatically.

```java
Set<Integer> numbers = new TreeSet<>();
numbers.add(5);
numbers.add(1);
numbers.add(10);

System.out.println(numbers); // Output: [1, 5, 10] (Always sorted)
```

## 3. `LinkedHashSet`

An intermediate option between `HashSet` and `TreeSet`.

*   **Structure:** Hash table + Doubly-linked list running through all entries.
*   **Ordering:** **Insertion Order**. Iterating through it returns elements in the order they were inserted.
*   **Performance:** Slightly slower than `HashSet` due to maintaining the linked list (`O(1)` operations).

```java
Set<String> insertionSet = new LinkedHashSet<>();
insertionSet.add("One");
insertionSet.add("Two");
insertionSet.add("Three");

System.out.println(insertionSet); // Output: [One, Two, Three] (Insertion order preserved)
```

## 4. Comparison Summary

| Implementation | Internal Structure | Ordering | Access/Add Complexity | Nulls? |
| :--- | :--- | :--- | :--- | :--- |
| **HashSet** | Hash Table | None | **O(1)** | Yes (1) |
| **TreeSet** | Red-Black Tree | **Sorted** | O(log n) | No |
| **LinkedHashSet** | Hash Table + Linked List | **Insertion** | O(1) (slightly slower) | Yes (1) |

### Choosing the Right Set
1.  **Default choice:** Use **`HashSet`**. It's the fastest.
2.  **Order matters?** Use `LinkedHashSet` if you want insertion order.
3.  **Sorting required?** Use `TreeSet` if you want elements sorted by value.
