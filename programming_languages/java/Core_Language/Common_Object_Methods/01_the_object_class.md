# The Object Class

In Java, the `java.lang.Object` class is the **root** of the class hierarchy. Every class you create or use implicitly extends `Object` directly or indirectly. This means that every object in Java inherits the methods defined in the `Object` class.

## 1. Why is there a Root Class?
Having a common root class ensures that every object in the Java ecosystem shares a basic set of behaviors. This allows for:
*   **Polymorphism:** You can write methods that accept `Object` and they will work with any type of object.
*   **Generic Programming:** Collections like `ArrayList` (before Generics) could store any object because they stored `Object` types.

## 2. Key Methods of the Object Class
The `Object` class defines several crucial methods. You often need to override these to provide meaningful behavior for your custom classes.

| Method | Description | Default Behavior |
| :--- | :--- | :--- |
| **`public String toString()`** | Returns a string representation of the object. | `ClassName@HashCodeHex` |
| **`public boolean equals(Object obj)`** | Checks if some other object is "equal to" this one. | Reference equality (`==`) |
| **`public int hashCode()`** | Returns a hash code value for the object. | Memory address based integer |
| **`protected Object clone()`** | Creates and returns a copy of this object. | Throws `CloneNotSupportedException` unless `Cloneable` |
| **`public final Class<?> getClass()`** | Returns the runtime class of this Object. | Returns `Class` metadata object |
| **`protected void finalize()`** | Called by GC before reclamation (Deprecated). | Does nothing |
| **`public final void wait()`** | Causes current thread to wait until notified. | Thread synchronization |
| **`public final void notify()`** | Wakes up a single thread waiting on this object. | Thread synchronization |
| **`public final void notifyAll()`** | Wakes up all threads waiting on this object. | Thread synchronization |

## 3. Using `Object` as a Type
Since `Object` is the parent of all classes, you can use it as a reference type for any object.

```java
Object obj1 = new String("Hello");
Object obj2 = new Integer(10);
Object obj3 = new Dog(); // Assuming Dog is a defined class

// Useful for generic handling
public void printObjectInfo(Object obj) {
    System.out.println("Class: " + obj.getClass().getName());
    System.out.println("String Ref: " + obj.toString());
}
```

*   **Limitation:** When you have a reference of type `Object`, you can only call methods defined in the `Object` class. To call methods specific to the actual object type (e.g., `Dog.bark()`), you must cast it back to its specific type.

```java
Object myDog = new Dog();
// myDog.bark(); // Compile-time error: Object class has no bark() method

if (myDog instanceof Dog) {
    Dog d = (Dog) myDog;
    d.bark(); // Works fine
}
```

---
In the following sections, we will dive deep into the most commonly overridden methods: `equals()`, `hashCode()`, `toString()`, and `clone()`.