# Bounded Type Parameters: Constraining Generic Types

While generics provide immense flexibility, sometimes you need to restrict the types that can be substituted for a type parameter. For example, you might want a generic method that operates on a list of numbers, but only if they are `Number` or a subclass of `Number`. **Bounded type parameters** allow you to impose such constraints, enhancing type safety and enabling access to methods of the bound types.

## 1. Why Bounded Types? Enabling Specific Operations

Without bounds, a type parameter (like `T`) is effectively treated as `Object`. This means you can only call methods available on `Object` itself (`equals()`, `hashCode()`, `toString()`).
If you need to perform operations specific to a certain class or interface (e.g., call `doubleValue()` on a `Number`, or `compareTo()` on a `Comparable`), you must bound the type parameter.

## 2. Upper Bounded Type Parameters (`extends`)

An upper-bounded type parameter specifies that the type argument must be `T` or a **subtype** (subclass or implementing interface) of `T`.

### Syntax
*   **For a class:** `class MyGenericClass<T extends SomeClass>`
*   **For a method:** `public <T extends SomeClass> void myGenericMethod(T arg)`

### Example: Restricting to `Number` and its Subclasses
```java
import java.util.List;
import java.util.ArrayList;

// A generic box that can only hold numbers (Integer, Double, etc.)
public class NumericBox<T extends Number> { // T must be Number or a subclass of Number
    private T number;

    public NumericBox(T number) { 
        this.number = number; 
    }

    public double getDoubleValue() {
        // Because T is bounded by Number, we can safely call Number's methods
        return number.doubleValue(); 
    }

    // A generic method that sums a list of any type that extends Number
    public static <U extends Number> double sumOfList(List<U> list) {
        double sum = 0.0;
        for (U num : list) {
            sum += num.doubleValue(); // Can call Number methods on 'num'
        }
        return sum;
    }

    public static void main(String[] args) {
        NumericBox<Integer> intBox = new NumericBox<>(10);
        System.out.println("Integer box value as double: " + intBox.getDoubleValue()); // Output: 10.0

        NumericBox<Double> doubleBox = new NumericBox<>(15.5);
        System.out.println("Double box value as double: " + doubleBox.getDoubleValue()); // Output: 15.5

        // NumericBox<String> stringBox = new NumericBox<>("hello"); // COMPILE-TIME ERROR: String is not a Number

        List<Integer> intList = Arrays.asList(1, 2, 3, 4, 5);
        System.out.println("Sum of integers in list: " + sumOfList(intList)); // Output: 15.0

        List<Double> doubleList = Arrays.asList(1.1, 2.2, 3.3);
        System.out.println("Sum of doubles in list: " + sumOfList(doubleList)); // Output: 6.6000000000000005
    }
}
```
*   **Key takeaway:** When you declare `T extends Number`, `T` can be `Number` itself or any class that extends `Number` (like `Integer`, `Double`, `Float`, etc.). Inside the generic code, `T` can be treated as a `Number`, allowing you to call `Number`'s methods.

### Multiple Bounds
A type parameter can have multiple bounds. This is useful when a type needs to implement several interfaces or extend a class and implement interfaces.
`public <T extends SomeClass & SomeInterface1 & SomeInterface2>`
*   **Rules:**
    *   If a class is specified, it must be the **first** in the `extends` list.
    *   Only **one** class can be specified, but multiple interfaces can be listed.
    *   (e.g., `T extends Number & Comparable<T>`)

## 3. Lower Bounded Type Parameters (`super`)

A lower-bounded type parameter specifies that the type argument must be `T` or a **supertype** of `T`. This is typically used when you want to **add** elements to a generic structure (a consumer of type `T`).

### Syntax
`public <T> void myGenericMethod(List<? super T> list)`

### Example: Copying from a source list to a destination list
```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;

public class ListUtils {
    // This method copies elements from a source list to a destination list.
    // The source (src) is a producer of elements (we read from it).
    // The destination (dest) is a consumer of elements (we write to it).
    public static <T> void copy(List<? extends T> src, List<? super T> dest) {
        // '? extends T' means src can be List<T> or List<SubtypeOfT>
        // '? super T' means dest can be List<T> or List<SupertypeOfT>
        for (T element : src) {
            dest.add(element); // OK: We can add 'T' to a List<? super T>
        }
    }

    public static void main(String[] args) {
        List<Integer> ints = Arrays.asList(1, 2, 3);
        List<Number> nums = new ArrayList<>();
        List<Object> objs = new ArrayList<>();

        // Copy Integers (subtype of Number) to a List of Numbers (supertype of Integer)
        copy(ints, nums); 
        System.out.println("Nums after copying ints: " + nums); // Output: Nums after copying ints: [1, 2, 3]

        // Copy Numbers (subtype of Object) to a List of Objects (supertype of Number)
        copy(nums, objs); 
        System.out.println("Objs after copying nums: " + objs); // Output: Objs after copying nums: [1, 2, 3]

        // copy(nums, ints); // COMPILE-TIME ERROR: List<Integer> is not a supertype of Number
                            // The destination list must be able to hold 'Number' or its supertypes.
    }
}
```
*   **Key takeaway for `<? super T>`:**
    *   **Can add:** You can add objects of type `T` (or any subtype of `T`) to a `List<? super T>`.
    *   **Cannot read:** When reading from a `List<? super T>`, you can only assume the elements are `Object` (because you don't know the specific supertype).

## 4. Unbounded Type Parameters (`<?>`)

An unbounded wildcard (`<?>`) specifies that the type argument can be any type. It is essentially equivalent to `<? extends Object>`.

### Syntax
`public void printList(List<?> list)`

### Use Cases
*   When the generic method's logic does not depend on the type parameter (e.g., `List.size()`, `List.clear()`).
*   When you are only reading elements from a generic structure, and treating them as `Object` is sufficient.

```java
public class Printer {
    public static void printList(List<?> list) { // Can take List<String>, List<Integer>, etc.
        for (Object elem : list) { // Elements are read as Object
            System.out.println(elem);
        }
        // list.add("new item"); // COMPILE-TIME ERROR: Cannot add elements (except null)
    }

    public static void main(String[] args) {
        List<String> strings = Arrays.asList("Apple", "Banana");
        List<Integer> integers = Arrays.asList(10, 20);

        printList(strings);
        printList(integers);
    }
}
```
*   **Limitation:** You generally **cannot add elements** to a `List<?>` (except `null`) because the compiler doesn't know the exact type that `?` represents, which could violate type safety.

## 5. The PECS Principle (Producer Extends, Consumer Super)

This mnemonic is a widely used guideline for remembering when to use `extends` or `super` wildcards, proposed by Joshua Bloch in "Effective Java".

*   **P**roducer **E**xtends: If your generic structure is a **producer** (it produces items for your code to read), use `<? extends T>`.
    *   You can *get* (read) `T` or its subtypes from it.
    *   You cannot *put* (write) `T` or its subtypes into it (except `null`).
*   **C**onsumer **S**uper: If your generic structure is a **consumer** (it consumes items your code puts into it), use `<? super T>`.
    *   You can *put* (write) `T` or its subtypes into it.
    *   You cannot *get* (read) `T` or its subtypes from it (only `Object`).

This principle is crucial for designing APIs that are both flexible (can work with a wide range of related types) and type-safe.

---

### Links to Topics:
*   [Generics Basics](01_generics_basics.md)
*   [Generic Classes and Interfaces](02_generic_classes_and_interfaces.md)
*   [Bounded Type Parameters](03_bounded_type_parameters.md)
*   [Wildcards](04_wildcards.md)
*   [Type Erasure](05_type_erasure.md)