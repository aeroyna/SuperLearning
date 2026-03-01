# Accessing Fields and Methods with Reflection

Once you have a `Class` object, Java Reflection allows you to dynamically discover and interact with the fields (variables) and methods of that class. This means you can get and set field values or call methods without knowing their names until runtime.

## 1. Accessing Fields

The `java.lang.reflect.Field` class represents a field of a class or an interface.

### Obtaining Fields
*   `clazz.getField(String name)`: Returns a `public` field matching the given name. Throws `NoSuchFieldException`.
*   `clazz.getFields()`: Returns an array of all `public` fields.
*   `clazz.getDeclaredField(String name)`: Returns any field (public, private, protected, default) declared *by this class* (not inherited). Throws `NoSuchFieldException`.
*   `clazz.getDeclaredFields()`: Returns an array of all fields declared *by this class*.

### Getting and Setting Field Values
```java
import java.lang.reflect.Field;

public class FieldAccess {
    private String privateField = "Private Value";
    public String publicField = "Public Value";

    public static void main(String[] args) throws Exception {
        MyObject obj = new MyObject();
        Class<?> clazz = obj.getClass();

        // 1. Accessing a Public Field
        Field publicField = clazz.getField("publicField");
        System.out.println("Public Field Name: " + publicField.getName());
        System.out.println("Public Field Value (before): " + publicField.get(obj)); // Get value
        publicField.set(obj, "New Public Value"); // Set value
        System.out.println("Public Field Value (after): " + publicField.get(obj));

        // 2. Accessing a Private Field
        Field privateField = clazz.getDeclaredField("privateField"); // Use getDeclaredField
        privateField.setAccessible(true); // Crucial: Override private access!
        System.out.println("Private Field Name: " + privateField.getName());
        System.out.println("Private Field Value (before): " + privateField.get(obj));
        privateField.set(obj, "New Private Value");
        System.out.println("Private Field Value (after): " + privateField.get(obj));
    }

    static class MyObject {
        private String privateField = "Private Value";
        public String publicField = "Public Value";
    }
}
// Output:
// Public Field Name: publicField
// Public Field Value (before): Public Value
// Public Field Value (after): New Public Value
// Private Field Name: privateField
// Private Field Value (before): Private Value
// Private Field Value (after): New Private Value
```

## 2. Invoking Methods

The `java.lang.reflect.Method` class represents a method of a class or an interface.

### Obtaining Methods
*   `clazz.getMethod(String name, Class<?>... parameterTypes)`: Returns a `public` method matching the name and parameter types. Throws `NoSuchMethodException`.
*   `clazz.getMethods()`: Returns an array of all `public` methods (inherited included).
*   `clazz.getDeclaredMethod(String name, Class<?>... parameterTypes)`: Returns any method declared *by this class*. Throws `NoSuchMethodException`.
*   `clazz.getDeclaredMethods()`: Returns an array of all methods declared *by this class*.

### Invoking Methods
*   `method.invoke(Object obj, Object... args)`: Invokes the underlying method represented by this `Method` object, on the specified `obj` with the specified `args`.

```java
import java.lang.reflect.Method;

public class MethodAccess {
    private void privateMethod(String msg) {
        System.out.println("Private Method: " + msg);
    }
    public String publicMethod(String prefix, int num) {
        return prefix + " " + num;
    }

    public static void main(String[] args) throws Exception {
        MyObject obj = new MyObject();
        Class<?> clazz = obj.getClass();

        // 1. Invoking a Public Method
        Method publicMethod = clazz.getMethod("publicMethod", String.class, int.class);
        Object result = publicMethod.invoke(obj, "Hello", 123); // Invoke on 'obj' with arguments
        System.out.println("Public Method Result: " + result);

        // 2. Invoking a Private Method
        Method privateMethod = clazz.getDeclaredMethod("privateMethod", String.class);
        privateMethod.setAccessible(true); // Crucial: Override private access!
        privateMethod.invoke(obj, "From Reflection"); // Invoke on 'obj' with arguments
    }

    static class MyObject {
        private void privateMethod(String msg) {
            System.out.println("Private Method: " + msg);
        }
        public String publicMethod(String prefix, int num) {
            return prefix + " " + num;
        }
    }
}
// Output:
// Public Method Result: Hello 123
// Private Method: From Reflection
```

## 3. Creating Objects (Constructors)

The `java.lang.reflect.Constructor` class represents a constructor of a class.

### Obtaining Constructors
*   `clazz.getConstructor(Class<?>... parameterTypes)`: Returns a `public` constructor.
*   `clazz.getConstructors()`: Returns an array of all `public` constructors.
*   `clazz.getDeclaredConstructor(Class<?>... parameterTypes)`: Returns any constructor declared *by this class*.
*   `clazz.getDeclaredConstructors()`: Returns an array of all constructors declared *by this class*.

### Instantiating Objects
*   `constructor.newInstance(Object... initargs)`: Creates a new instance of the class.

```java
import java.lang.reflect.Constructor;

public class ConstructorAccess {
    String message;
    private int value;

    public ConstructorAccess(String msg) {
        this.message = msg;
    }

    private ConstructorAccess(String msg, int val) {
        this.message = msg;
        this.value = val;
    }

    public static void main(String[] args) throws Exception {
        Class<?> clazz = MyObjectWithConstructor.class;

        // 1. Using a Public Constructor
        Constructor<?> publicCtor = clazz.getConstructor(String.class);
        MyObjectWithConstructor obj1 = (MyObjectWithConstructor) publicCtor.newInstance("Public Hello");
        System.out.println("Obj1 message: " + obj1.message);

        // 2. Using a Private Constructor
        Constructor<?> privateCtor = clazz.getDeclaredConstructor(String.class, int.class);
        privateCtor.setAccessible(true); // Override private access
        MyObjectWithConstructor obj2 = (MyObjectWithConstructor) privateCtor.newInstance("Private Secret", 42);
        System.out.println("Obj2 message: " + obj2.message + ", value: " + obj2.value);
    }

    static class MyObjectWithConstructor {
        String message;
        private int value;

        public MyObjectWithConstructor(String msg) {
            this.message = msg;
        }

        private MyObjectWithConstructor(String msg, int val) {
            this.message = msg;
            this.value = val;
        }
    }
}
// Output:
// Obj1 message: Public Hello
// Obj2 message: Private Secret, value: 42
```

## 4. `setAccessible(true)`: Breaking Encapsulation

The `setAccessible(true)` method is crucial for accessing `private` fields and methods. It overrides Java's access control mechanisms.
*   **Warning:** Use this with extreme caution. It breaks encapsulation and can lead to unexpected behavior or security vulnerabilities. It's generally reserved for testing frameworks, serialization, or highly specialized library code.
*   In some secure environments (e.g., applets, certain security managers), `setAccessible(true)` might throw a `SecurityException`.
*   Since Java 9, strong encapsulation is enforced. Accessing private members of *JDK internal classes* will result in warnings or errors, even with `setAccessible(true)`, unless JVM flags are used.

Reflection's ability to manipulate internal state and behavior at runtime is powerful but comes with significant responsibilities regarding code stability, performance, and security.