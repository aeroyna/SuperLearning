# Enum Methods and Constructors

Enums in Java are far more powerful than simple sets of constants. Because each enum constant is an instance of its enum type, you can give enums fields, methods, and constructors, making them behave like specialized classes.

## 1. Enum Fields (Instance Variables)

Each enum constant can have its own specific data associated with it. You declare fields within the enum body, just like in a class.

### Example: Traffic Light Enum with Duration
```java
public enum TrafficLight {
    RED(30),   // Each constant calls a constructor
    YELLOW(5),
    GREEN(45);

    private final int duration; // Enum fields are implicitly final

    // Constructor (implicitly private)
    TrafficLight(int duration) {
        this.duration = duration;
    }

    // Getter method
    public int getDuration() {
        return duration;
    }

    public static void main(String[] args) {
        System.out.println("Red light duration: " + TrafficLight.RED.getDuration() + " seconds");
        // Output: Red light duration: 30 seconds
    }
}
```

## 2. Enum Constructors

*   **Implicitly `private`:** Enum constructors are always `private` (or package-private if no modifier is specified). You cannot declare them as `public` or `protected`. This ensures that you cannot create new enum instances arbitrarily, maintaining the fixed set of constants.
*   **Called by Constants:** The constructor is invoked once for each enum constant when the enum type is first loaded.

```java
// Constructor for TrafficLight enum above:
// private TrafficLight(int duration) { // implicitly private
//     this.duration = duration;
// }
```

## 3. Enum Methods

Enums can have regular methods, just like classes. This allows you to encapsulate behavior directly within the enum.

### Example: Basic Enum with Method
```java
public enum Operation {
    ADD, SUBTRACT, MULTIPLY, DIVIDE;

    public int perform(int x, int y) {
        switch (this) { // 'this' refers to the enum constant
            case ADD: return x + y;
            case SUBTRACT: return x - y;
            case MULTIPLY: return x * y;
            case DIVIDE: return x / y;
            default: throw new AssertionError("Unknown operation: " + this);
        }
    }

    public static void main(String[] args) {
        System.out.println(Operation.ADD.perform(10, 5)); // Output: 15
        System.out.println(Operation.DIVIDE.perform(10, 5)); // Output: 2
    }
}
```

## 4. Constant-Specific Class Body (Constant-Specific Method Implementations)

This powerful feature allows each enum constant to provide its own specific implementation for an enum method. This is a form of **polymorphism** within enums.

### Example: Operation Enum with Abstract Method
```java
public enum Operation {
    ADD { // Constant-specific class body
        @Override
        public int perform(int x, int y) { return x + y; }
    },
    SUBTRACT {
        @Override
        public int perform(int x, int y) { return x - y; }
    },
    MULTIPLY {
        @Override
        public int perform(int x, int y) { return x * y; }
    },
    DIVIDE {
        @Override
        public int perform(int x, int y) { 
            if (y == 0) throw new IllegalArgumentException("Cannot divide by zero.");
            return x / y; 
        }
    }; // Semicolon is required if methods/fields follow constants

    // Abstract method that each constant must implement
    public abstract int perform(int x, int y);

    public static void main(String[] args) {
        System.out.println(Operation.ADD.perform(10, 5));     // Output: 15
        System.out.println(Operation.SUBTRACT.perform(10, 5)); // Output: 5
        System.out.println(Operation.DIVIDE.perform(10, 0)); // Throws IllegalArgumentException
    }
}
```
*   In this pattern, the `Operation` enum itself declares an `abstract` method `perform`. Each constant then provides its own implementation of this abstract method. This is much cleaner and more extensible than a large `switch` statement in a single `perform` method.

## 5. Implementing Interfaces

Enums can implement interfaces. This allows enums to participate in polymorphic designs alongside regular classes.

### Example: Enum Implementing an Interface
```java
interface Displayable {
    String getDisplayName();
}

public enum Status implements Displayable {
    ACTIVE("Active User"),
    INACTIVE("Inactive User"),
    PENDING("Pending Approval");

    private final String displayName;

    Status(String displayName) {
        this.displayName = displayName;
    }

    @Override
    public String getDisplayName() {
        return displayName;
    }

    public static void main(String[] args) {
        for (Status s : Status.values()) {
            System.out.println(s.name() + ": " + s.getDisplayName());
        }
    }
}
```

## 6. Best Practices
*   **Use for Fixed Sets:** Enums are best for fixed sets of constants that do not change frequently.
*   **Encapsulate Behavior:** Store related data and behavior directly within the enum.
*   **Constant-Specific Implementations:** Leverage polymorphism within enums to avoid bulky `switch` statements.

These advanced features make enums a powerful tool for creating robust, type-safe, and self-documenting code in Java.