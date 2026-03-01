# Generic Classes and Interfaces

Generics allow you to define classes and interfaces with type parameters. This means you can create a single blueprint that can operate on different data types, providing compile-time type safety and code reusability.

## 1. Generic Classes

A generic class is declared with one or more type parameters. These type parameters act as placeholders for actual types that will be provided when an object of the generic class is created.

### Syntax
```java
class MyGenericClass<T1, T2, ...> {
    // T1, T2, ... are type parameters
}
```

### Example: A Simple `Box` Class
Let's create a `Box` that can hold any single item.

```java
public class Box<T> { // T is the type parameter
    private T item; // T is used as the type of the 'item' field

    public void setItem(T item) {
        this.item = item;
    }

    public T getItem() {
        return item;
    }

    public static void main(String[] args) {
        // Create a Box to hold a String
        Box<String> stringBox = new Box<>(); // Using diamond operator (Java 7+)
        stringBox.setItem("Hello Generics");
        String myString = stringBox.getItem(); // No cast needed
        System.out.println(myString);

        // Create a Box to hold an Integer
        Box<Integer> integerBox = new Box<>();
        integerBox.setItem(12345);
        Integer myInteger = integerBox.getItem(); // No cast needed
        System.out.println(myInteger);

        // Attempting to put wrong type will be a compile-time error
        // stringBox.setItem(10); // Compile-time error!
    }
}
```

### Multiple Type Parameters
A generic class can have more than one type parameter.
```java
public class Pair<K, V> { // K for Key, V for Value
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }

    public static void main(String[] args) {
        Pair<String, Integer> p1 = new Pair<>("Age", 30);
        System.out.println("Key: " + p1.getKey() + ", Value: " + p1.getValue());

        Pair<Integer, String> p2 = new Pair<>(1, "One");
        System.out.println("Key: " + p2.getKey() + ", Value: " + p2.getValue());
    }
}
```

## 2. Generic Interfaces

Similar to generic classes, you can define interfaces with type parameters.

### Syntax
```java
interface MyGenericInterface<T> {
    T performAction(T data);
}
```

### Example: A `Repository` Interface
```java
// Generic interface for a basic data repository
interface Repository<T, ID> { // T for Entity type, ID for Identifier type
    T findById(ID id);
    void save(T entity);
    void delete(T entity);
    // ... more generic methods
}

// A concrete implementation for User entities with Integer IDs
class UserRepository implements Repository<User, Integer> {
    @Override
    public User findById(Integer id) {
        System.out.println("Finding user with ID: " + id);
        return new User("TestUser"); // Placeholder
    }

    @Override
    public void save(User entity) {
        System.out.println("Saving user: " + entity.getName());
    }

    @Override
    public void delete(User entity) {
        System.out.println("Deleting user: " + entity.getName());
    }
    
    // Nested helper class for example
    static class User {
        String name;
        User(String name) { this.name = name; }
        String getName() { return name; }
    }

    public static void main(String[] args) {
        UserRepository userRepo = new UserRepository();
        User user = userRepo.findById(123);
        userRepo.save(user);
    }
}
```

## 3. Type Parameter Naming Conventions
As seen in the Generics Basics chapter, it's a good practice to follow standard naming conventions for type parameters to improve readability (e.g., `T`, `E`, `K`, `V`, `N`, `S`, `U`).

## 4. Generic Methods

You can also define generic methods, which can be part of a generic class, a non-generic class, or an interface. The type parameter is declared *before* the return type of the method.

### Syntax
```java
// <T> is the type parameter declaration for the method
public <T> void genericMethod(T data) {
    // ...
}
```

### Example: A Generic Utility Method
```java
public class GenericUtils {
    // This method can print any type of object
    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.print(element + " ");
        }
        System.out.println();
    }

    // This method returns the middle element of an array
    public static <T> T getMiddleElement(T[] array) {
        if (array == null || array.length == 0) {
            return null;
        }
        return array[array.length / 2];
    }

    public static void main(String[] args) {
        Integer[] intArray = {1, 2, 3, 4, 5};
        String[] stringArray = {"A", "B", "C"};

        printArray(intArray);    // Output: 1 2 3 4 5
        printArray(stringArray); // Output: A B C

        Integer middleInt = getMiddleElement(intArray);
        System.out.println("Middle Integer: " + middleInt); // Output: Middle Integer: 3

        String middleString = getMiddleElement(stringArray);
        System.out.println("Middle String: " + middleString); // Output: Middle String: B
    }
}
```
Generic methods are incredibly useful for writing flexible utility functions that can operate on different types.