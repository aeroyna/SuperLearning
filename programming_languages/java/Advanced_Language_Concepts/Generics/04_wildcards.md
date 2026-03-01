# Wildcards

Wildcards (`?`) in Java Generics provide more flexibility in handling generic types. They allow you to define generic types with unknown or partially known type arguments, which is essential for writing versatile APIs.

## 1. The Unbounded Wildcard (`<?>`)

*   **Syntax:** `<?>`
*   **Meaning:** Represents an unknown type. It means "any type."
*   **Use Cases:**
    *   When the method's logic does not depend on the type parameter (e.g., `List.size()`, `List.clear()`).
    *   When you are only reading data from a generic collection, and `Object` is sufficient as the type of the elements.
    *   When defining a method that can operate on a generic type regardless of its specific type argument.

### Example
```java
import java.util.List;
import java.util.ArrayList;

public class UnboundedWildcard {
    public static void printList(List<?> list) { // Can accept List<String>, List<Integer>, etc.
        for (Object elem : list) { // Elements are read as Object
            System.out.println(elem);
        }
        // list.add("item"); // Compile-time error: Cannot add elements (except null)
        // list.add(new Object()); // Still an error
    }

    public static void main(String[] args) {
        List<String> names = new ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        printList(names); // Output: Alice, Bob

        List<Integer> numbers = new ArrayList<>();
        numbers.add(1);
        numbers.add(2);
        printList(numbers); // Output: 1, 2
    }
}
```
*   **Limitation:** You generally **cannot add elements** to a `List<?>` (except `null`) because the compiler doesn't know the exact type `?` represents. This prevents violating type safety.

## 2. Upper Bounded Wildcard (`<? extends T>`)

*   **Syntax:** `<? extends T>`
*   **Meaning:** Represents an unknown type that is `T` or a **subtype** of `T`.
*   **Use Cases:**
    *   When you want to **read** values from a generic collection (producer).
    *   It allows polymorphism: a `List<? extends Number>` can hold `List<Integer>`, `List<Double>`, etc.

### Example
```java
import java.util.List;
import java.util.ArrayList;

class Shape {}
class Circle extends Shape {}
class Square extends Shape {}

public class UpperBoundedWildcard {
    // This method can draw any list of Shapes or their subclasses
    public static void drawAll(List<? extends Shape> shapes) { // Producer: Reads Shapes
        for (Shape s : shapes) { // Can read elements as Shape (or its subtypes)
            // s.draw(); // Assuming Shape has a draw method
            System.out.println("Drawing: " + s.getClass().getSimpleName());
        }
        // shapes.add(new Circle()); // Compile-time error: Cannot add elements (except null)
    }

    public static void main(String[] args) {
        List<Shape> myShapes = new ArrayList<>();
        myShapes.add(new Circle());
        myShapes.add(new Square());
        drawAll(myShapes); // Output: Drawing: Circle, Drawing: Square

        List<Circle> myCircles = new ArrayList<>();
        myCircles.add(new Circle());
        drawAll(myCircles); // Output: Drawing: Circle (List<Circle> is compatible with List<? extends Shape>)

        // List<Object> objects = new ArrayList<>();
        // drawAll(objects); // Compile-time error: Object is not a subtype of Shape
    }
}
```
*   **Limitation:** Similar to `<?>`, you generally **cannot add elements** to a `List<? extends T>` (except `null`) because the compiler doesn't know the specific subtype of `T` it holds.

## 3. Lower Bounded Wildcard (`<? super T>`)

*   **Syntax:** `<? super T>`
*   **Meaning:** Represents an unknown type that is `T` or a **supertype** of `T`.
*   **Use Cases:**
    *   When you want to **add/write** values to a generic collection (consumer).
    *   It allows polymorphism: a `List<? super Integer>` can hold `List<Integer>`, `List<Number>`, `List<Object>`, etc.

### Example
```java
import java.util.List;
import java.util.ArrayList;

public class LowerBoundedWildcard {
    // This method adds an Integer to any list that can hold Integers or their supertypes
    public static void addInteger(List<? super Integer> list) { // Consumer: Writes Integers
        list.add(10); // OK to add an Integer
        list.add(20); // OK to add another Integer
        // Integer num = list.get(0); // Compile-time error: Can't read a specific type (only Object)
    }

    public static void main(String[] args) {
        List<Integer> ints = new ArrayList<>();
        addInteger(ints); // List<Integer> is compatible with List<? super Integer>
        System.out.println("Ints: " + ints); // Output: Ints: [10, 20]

        List<Number> nums = new ArrayList<>();
        addInteger(nums); // List<Number> is compatible
        System.out.println("Nums: " + nums); // Output: Nums: [10, 20]

        List<Object> objs = new ArrayList<>();
        addInteger(objs); // List<Object> is compatible
        System.out.println("Objs: " + objs); // Output: Objs: [10, 20]
    }
}
```
*   **Limitation:** You generally **cannot read** elements as a specific type from a `List<? super T>` (only as `Object`) because the compiler doesn't know the exact supertype of `T` it holds.

## 4. The PECS Principle (Producer Extends, Consumer Super)

This mnemonic is a widely used guideline for deciding when to use `extends` or `super` wildcards:

*   **Producer Extends:** If your generic structure is a **producer** (it produces items for your code to read), use `<? extends T>`. (You can only `get` from it, not `put`).
*   **Consumer Super:** If your generic structure is a **consumer** (it consumes items your code puts into it), use `<? super T>`. (You can only `put` into it, not `get` a specific type).

### Example applying PECS
```java
// Method that copies elements from source to destination
public static <T> void copy(List<? extends T> src, List<? super T> dest) {
    // src is a PRODUCER: We read elements from src.
    // dest is a CONSUMER: We write elements to dest.
    for (T element : src) {
        dest.add(element); // This is type-safe
    }
}
```

Understanding wildcards and the PECS principle is key to designing robust and flexible generic APIs in Java, especially when working with the Collections Framework.