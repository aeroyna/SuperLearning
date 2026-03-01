# String Pool

The String Pool (also known as the String Intern Pool or String Literal Pool) is a special memory area within the Java Heap that stores unique string literals. Its purpose is to optimize memory usage by ensuring that identical string literals share the same memory location.

## 1. How the String Pool Works

1.  **Creation of String Literals:** When you create a string using a literal (e.g., `String s = "hello";`), the JVM first checks if that string already exists in the String Pool.
    *   If it exists, the JVM simply returns a reference to that existing string object.
    *   If it does not exist, the JVM creates a new string object in the pool and returns a reference to it.
2.  **Creation with `new String()`:** When you create a string using the `new` keyword (e.g., `String s = new String("hello");`), a new object is always created in the Heap, outside the String Pool. If the literal "hello" doesn't exist in the pool, it might also create a copy there.

## 2. String Literals vs. `new String()`

This distinction is crucial for understanding string equality in Java.

### Example
```java
String s1 = "Hello";      // s1 refers to "Hello" in the String Pool
String s2 = "Hello";      // s2 also refers to the SAME "Hello" object in the Pool
String s3 = new String("Hello"); // s3 refers to a NEW "Hello" object in the Heap (not in the pool by default)
String s4 = new String("Hello"); // s4 refers to another NEW "Hello" object in the Heap
String s5 = "World";      // s5 refers to "World" in the String Pool
```

### Comparing Strings ( `==` vs. `.equals()` )

1.  **`==` Operator:** Checks for **reference equality** (do both variables point to the exact same object in memory?).

    ```java
    System.out.println(s1 == s2); // true (both point to the same object in the pool)
    System.out.println(s1 == s3); // false (s1 in pool, s3 in heap)
    System.out.println(s3 == s4); // false (s3 and s4 are different objects in heap)
    ```

2.  **`.equals()` Method:** Checks for **logical equality** (do both string objects have the same sequence of characters?).

    ```java
    System.out.println(s1.equals(s2)); // true
    System.out.println(s1.equals(s3)); // true
    System.out.println(s3.equals(s4)); // true
    ```
    **Rule of Thumb:** Always use `.equals()` to compare string content. The `==` operator for strings is almost always a bug.

## 3. The `intern()` Method

The `intern()` method can be called on any `String` object.
*   If the String Pool already contains a string equal to this `String` object (as determined by the `equals()` method), then the string from the pool is returned.
*   Otherwise, this `String` object is added to the pool and a reference to it is returned.

### Example
```java
String s3 = new String("Hello"); // Creates "Hello" in Heap, maybe also in Pool if not there
String s1 = "Hello";             // s1 refers to "Hello" in Pool

System.out.println(s1 == s3);          // false
System.out.println(s1 == s3.intern()); // true (s3.intern() returns the pooled "Hello")
```

## 4. Location of the String Pool

*   **Prior to Java 7:** The String Pool was located in the **PermGen space** of the JVM (a part of the Method Area). PermGen had a fixed size and could lead to `OutOfMemoryError`s if too many unique strings were interned.
*   **Java 7 and later:** The String Pool was moved to the **Heap space**. This makes it dynamically adjustable and subject to garbage collection, reducing `OutOfMemoryError`s related to the pool.

## 5. Summary
The String Pool is an optimization mechanism that leverages the immutability of `String` objects. It helps conserve memory and improve performance, especially when dealing with many identical string literals. Understanding `==` vs. `.equals()` and the role of `intern()` is vital for correct and efficient string handling in Java.
