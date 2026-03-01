# Reflection Basics

Reflection is a powerful feature in Java that allows an executing Java program to **examine or modify its own structure and behavior at runtime**. This includes inspecting classes, interfaces, fields, and methods, and even invoking methods or creating new objects without knowing their names at compile time.

## 1. What is Reflection?

*   **Introspection:** The ability to inspect the metadata and structure of classes, objects, and their members at runtime.
*   **Manipulation:** The ability to instantiate objects, invoke methods, and modify field values dynamically at runtime.

### Why is it called "Reflection"?
The term "reflection" means looking back at oneself. In programming, it refers to a program's ability to examine and manipulate its own internal structure.

## 2. The `Class` Object: The Gateway to Reflection

Every object in Java has a `Class` object, which is an instance of the `java.lang.Class` class. The `Class` object represents the metadata of a class or interface (e.g., its name, fields, methods, constructors, interfaces it implements, superclass).

### Ways to Obtain a `Class` Object

1.  **Using `Class.forName(String className)`:**
    *   Most common for dynamic class loading. Throws `ClassNotFoundException`.
    *   Used when the class name is known at runtime but not at compile time.
    ```java
    try {
        Class<?> myClass = Class.forName("java.lang.String");
        System.out.println(myClass.getName()); // Output: java.lang.String
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
    ```

2.  **Using `Object.getClass()`:**
    *   If you already have an instance of an object.
    ```java
    String s = "Hello";
    Class<?> stringClass = s.getClass();
    System.out.println(stringClass.getName()); // Output: java.lang.String
    ```

3.  **Using `.class` Literal:**
    *   If you know the class at compile time (most common for compile-time known classes).
    ```java
    Class<?> integerClass = Integer.class;
    System.out.println(integerClass.getName()); // Output: java.lang.Integer
    ```

4.  **For Primitives and `void`:**
    ```java
    Class<?> intPrimitive = int.class;
    Class<?> voidType = void.class;
    ```

## 3. Basic Introspection with `Class`

Once you have a `Class` object, you can discover a lot about the class it represents.

```java
Class<?> stringClass = String.class;

// Get class name
System.out.println("Name: " + stringClass.getName());           // java.lang.String
System.out.println("Simple Name: " + stringClass.getSimpleName()); // String

// Get superclass
Class<?> superClass = stringClass.getSuperclass();
System.out.println("Superclass: " + superClass.getName());      // java.lang.Object

// Get interfaces implemented
Class<?>[] interfaces = stringClass.getInterfaces();
for (Class<?> iface : interfaces) {
    System.out.println("Interface: " + iface.getName());        // java.io.Serializable, java.lang.CharSequence, java.lang.Comparable
}

// Check if it's an interface, primitive, array
System.out.println("Is interface? " + stringClass.isInterface()); // false
System.out.println("Is primitive? " + int.class.isPrimitive()); // true
System.out.println("Is array? " + String[].class.isArray()); // true
```

## 4. Use Cases of Reflection

*   **Frameworks (e.g., Spring, Hibernate, JUnit):**
    *   **Dependency Injection:** Spring uses reflection to discover components and inject dependencies.
    *   **ORM (Object-Relational Mapping):** Hibernate maps Java objects to database tables by inspecting class fields.
    *   **Testing Frameworks:** JUnit uses reflection to find and invoke test methods (`@Test`).
*   **IDEs:** Use reflection for code completion, refactoring, and debugging.
*   **Debugging Tools:** Examine object states at runtime.
*   **Dynamic Class Loading:** Loading classes based on configuration (e.g., plugin architectures).
*   **Custom Annotations Processing:** As seen in the previous chapter, custom annotations are processed via reflection at runtime.

## 5. Drawbacks and Cautions

While powerful, Reflection comes with costs:
*   **Performance Overhead:** Reflection involves more processing than direct code calls, making it slower.
*   **Security Issues:** Reflection allows bypassing access restrictions (`private` members), which can compromise security and encapsulation.
*   **Increased Complexity:** Code using reflection can be harder to read, debug, and maintain.
*   **Compile-time Safety Loss:** Errors that would normally be caught at compile time (e.g., `NoSuchMethodException`) now manifest as runtime exceptions.
*   **`NullPointerException`:** More likely if not handled carefully, as you deal with `Object` types.

Reflection should be used judiciously, typically by framework developers, not in everyday application logic.