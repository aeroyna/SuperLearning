# Generics Basics

## 1. What are Generics?

Generics, introduced in **Java 5**, allow you to write classes, interfaces, and methods that operate on types that are specified as **parameters** when the class, interface, or method is instantiated. They provide **compile-time type safety** without sacrificing flexibility.

### Analogy
Think of a **blueprint** for a container.
*   **Without Generics:** The blueprint says "container for anything." You put a `String` in, you get an `Object` out, and you hope it's a `String`.
*   **With Generics:** The blueprint says "container for only `String`s." You try to put an `Integer` in, the blueprint (compiler) stops you. When you get something out, you know it's a `String`.

## 2. Why Generics? The Problem Before Java 5

Before Java 5, collections stored objects of type `java.lang.Object`.

```java
import java.util.ArrayList;

ArrayList list = new ArrayList(); // No type specified
list.add("Hello");             // Add a String
list.add(123);                 // Add an Integer (auto-boxed to Object)

String s = (String) list.get(0); // Requires a cast, might fail at runtime
// String s2 = (String) list.get(1); // RUNTIME ERROR: ClassCastException
```

**Problems with pre-Generics code:**
1.  **No Type Safety:** The compiler cannot verify the type of objects being added to the collection.
2.  **Verbose & Error-Prone Casting:** Explicit casting is required when retrieving objects, which is boilerplate and can lead to `ClassCastException` at runtime if the wrong type is stored.

## 3. The Solution: Generics

Generics solve these problems by providing type-checking at **compile time**.

```java
import java.util.ArrayList;

// Declare ArrayList to hold only String objects
ArrayList<String> stringList = new ArrayList<String>(); 
stringList.add("Hello");
// stringList.add(123); // COMPILE-TIME ERROR: Incompatible types! Type safety enforced.

String s = stringList.get(0); // No cast needed!

// The Diamond Operator (<>) - Java 7+
// The compiler can infer the type argument from the context
ArrayList<String> stringList2 = new ArrayList<>(); 
```

**Benefits of Generics:**
1.  **Type Safety:** Catches `ClassCastException`s at compile time rather than runtime.
2.  **Eliminate Casts:** Removes the need for explicit type casting.
3.  **Code Reusability:** Allows writing algorithms and data structures that work with different types without sacrificing type safety.

## 4. Basic Generic Syntax

### Type Parameters
*   Generic types use **type parameters** (also called type variables) enclosed in angle brackets `<>`.
*   Commonly used type parameter names (by convention):
    *   `E` - Element (used extensively by the Java Collections Framework)
    *   `K` - Key
    *   `V` - Value
    *   `N` - Number
    *   `T` - Type (most general)
    *   `S`, `U` - Second, third, etc. type

### Generic Class Example (Conceptual, detailed in next chapter)
```java
// T is a type parameter
public class Box<T> {
    private T content;

    public void setContent(T content) {
        this.content = content;
    }

    public T getContent() {
        return content;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.setContent("My String");
String myString = stringBox.getContent(); // Type-safe, no cast

Box<Integer> integerBox = new Box<>();
integerBox.setContent(123);
Integer myInteger = integerBox.getContent(); // Type-safe, no cast
```

---

## 5. Raw Types (Avoid!)

A raw type is a generic type used without type arguments (e.g., `ArrayList` instead of `ArrayList<String>`).
```java
ArrayList list = new ArrayList(); // Raw type
list.add("abc");
list.add(123); // No compile-time error here!

ArrayList<String> stringList = list; // WARNING: Unchecked assignment
// String s = stringList.get(1); // ClassCastException at runtime!
```
*   Raw types are primarily for **backward compatibility** with pre-Java 5 code.
*   Using raw types loses the benefits of generics and can lead to runtime errors. Always use parameterized types when working with generics.

This basic understanding sets the stage for diving into more complex generic concepts.