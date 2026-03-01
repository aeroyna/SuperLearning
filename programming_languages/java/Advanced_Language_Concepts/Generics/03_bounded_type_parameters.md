# Bounded Type Parameters

By default, a generic type parameter (like `T`) can be replaced by any class type. Bounded type parameters allow you to restrict the types that can be used as type arguments for a generic type.

## 1. Upper Bounded Wildcards (`<? extends T>`)

An upper-bounded type parameter specifies that the type argument must be `T` or a subclass of `T`. This is useful for restricting input (reading from a generic structure).

### Syntax
`class MyGenericClass<T extends SomeClass>`
`public <T extends SomeClass> void myGenericMethod(T arg)`

### Example: Restricting to `Number` and its Subclasses
```java
public class NumericBox<T extends Number> { // T must be Number or a subclass (Integer, Double, etc.)
    private T number;

    public NumericBox(T number) { this.number = number; }

    public double doubleValue() {
        return number.doubleValue(); // Can call Number methods
    }

    // Generic method with an upper bound
    public static <U extends Number> double sumOfList(java.util.List<U> list) {
        double sum = 0.0;
        for (U num : list) {
            sum += num.doubleValue(); // Can call Number methods
        }
        return sum;
    }

    public static void main(String[] args) {
        NumericBox<Integer> intBox = new NumericBox<>(10);
        System.out.println(intBox.doubleValue()); // Output: 10.0

        NumericBox<Double> doubleBox = new NumericBox<>(15.5);
        System.out.println(doubleBox.doubleValue()); // Output: 15.5

        // NumericBox<String> stringBox = new NumericBox<>("hello"); // Compile-time error: String is not a Number

        java.util.List<Integer> intList = java.util.Arrays.asList(1, 2, 3);
        System.out.println("Sum of integers: " + sumOfList(intList)); // Output: 6.0

        java.util.List<Double> doubleList = java.util.Arrays.asList(1.1, 2.2, 3.3);
        System.out.println("Sum of doubles: " + sumOfList(doubleList)); // Output: 6.6
    }
}
```
*   **Key takeaway:** When you have an upper bound `T extends Number`, you can call methods of the `Number` class on objects of type `T`.

### Multiple Bounds
A type parameter can have multiple bounds:
`public <T extends SomeClass & SomeInterface>`
*   Only one class can be specified, but multiple interfaces.
*   The class bound must be specified first.

## 2. Lower Bounded Wildcards (`<? super T>`)

A lower-bounded type parameter specifies that the type argument must be `T` or a superclass of `T`. This is useful for restricting output (writing to a generic structure).

### Syntax
`public <T> void myGenericMethod(List<? super T> list)`

### Example: Copying from one list to another
```java
public class ListUtils {
    // This method copies elements from a source list to a destination list.
    // ? extends T: We can read elements of type T or its subtypes from 'src'.
    // ? super T: We can write elements of type T or its supertypes to 'dest'.
    public static <T> void copy(java.util.List<? extends T> src, java.util.List<? super T> dest) {
        for (T element : src) {
            dest.add(element); // OK to add T to a List<? super T>
        }
    }

    public static void main(String[] args) {
        java.util.List<Integer> ints = java.util.Arrays.asList(1, 2, 3);
        java.util.List<Number> nums = new java.util.ArrayList<>();
        java.util.List<Object> objs = new java.util.ArrayList<>();

        copy(ints, nums); // Copy Integers to a List of Numbers (Integer is subtype of Number)
        System.out.println("Nums: " + nums); // Output: Nums: [1, 2, 3]

        copy(nums, objs); // Copy Numbers to a List of Objects (Number is subtype of Object)
        System.out.println("Objs: " + objs); // Output: Objs: [1, 2, 3]

        // copy(nums, ints); // Compile-time error: List<Integer> is not a supertype of Number
    }
}
```
*   **Key takeaway:** When you have a lower bound `? super T`, you can add objects of type `T` (or its subtypes) to the list. You cannot read a `T` from the list with certainty (you'd get an `Object`).

## 3. Unbounded Wildcards (`<?>`)

An unbounded wildcard specifies that the type argument can be any type. It's equivalent to `<? extends Object>`.

### Syntax
`public void printList(List<?> list)`

### Use Cases
*   When the method's logic does not depend on the type parameter (e.g., `List.size()`).
*   When you are only reading from a generic structure and `Object` is sufficient, or writing `null`.

```java
public class Printer {
    public static void printList(java.util.List<?> list) { // Can take List<String>, List<Integer>, etc.
        for (Object elem : list) { // Elements are read as Object
            System.out.println(elem);
        }
    }

    public static void main(String[] args) {
        java.util.List<String> strings = java.util.Arrays.asList("A", "B", "C");
        java.util.List<Integer> integers = java.util.Arrays.asList(1, 2, 3);

        printList(strings);
        printList(integers);

        // list.add("new item"); // Compile-time error: cannot add elements (except null)
    }
}
```

## 4. PECS Principle (Producer Extends, Consumer Super)

This mnemonic helps remember when to use upper and lower bounds:
*   **Producer Extends:** If your generic structure is a **producer** (you read items from it), use `<? extends T>`.
*   **Consumer Super:** If your generic structure is a **consumer** (you write items to it), use `<? super T>`.

This helps in designing APIs that are flexible and type-safe.