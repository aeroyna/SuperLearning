# Methods and Encapsulation: Controlling Behavior and Data Integrity

Methods are the actions or behaviors that objects can perform. Encapsulation is a fundamental OOP principle that dictates how data and the methods operating on that data are bundled together and how access to that data is controlled. This chapter dives into the mechanics of methods and the crucial role of encapsulation in robust software design.

## 1. Methods: Defining Object Behaviors

A method is a block of code that performs a specific task. It provides a way to interact with an object's state and trigger its actions.

### Method Declaration Syntax: The Anatomy of a Method
```java
// Access_Modifier [Static_Modifier] [Final_Modifier] Return_Type Method_Name(Parameter_List) [throws Exception_List] {
//     // Method body: statements that perform the task
//     return value; // (if Return_Type is not void)
// }
```
*   **`Access_Modifier`:** (e.g., `public`, `private`, `protected`, default). Controls visibility.
*   **`Static_Modifier`:** (optional `static`). If present, the method belongs to the class itself, not an object.
*   **`Final_Modifier`:** (optional `final`). If present, the method cannot be overridden by subclasses.
*   **`Return_Type`:** The data type of the value the method will return. Use `void` if the method does not return any value.
*   **`Method_Name`:** Follows `camelCase` convention (e.g., `calculateTotal`, `getName`).
*   **`Parameter_List`:** A comma-separated list of `type name` pairs. Methods can have zero or more parameters. These are local variables initialized with values passed from the caller.
*   **`throws Exception_List`:** (optional `throws`). Declares checked exceptions that the method might throw but doesn't handle itself.
*   **Method Body:** The actual code block enclosed in curly braces `{}`.

### Example
```java
public class Calculator {
    // A method that adds two integers and returns their sum
    public int add(int a, int b) { // public, returns int, named add, takes two int parameters
        int sum = a + b; // Method body
        return sum;      // Returns an int value
    }

    // A method that prints a message and returns nothing (void)
    private void displayMessage(String message) { // private, returns nothing, takes one String parameter
        System.out.println("Message: " + message);
    }

    public void showCalculationResult(int x, int y) {
        int result = add(x, y); // Calling another method within the same class
        displayMessage("The sum of " + x + " and " + y + " is: " + result);
    }
}
```

### Calling Methods: Interacting with Objects
Methods are invoked using the dot operator (`.`) on an object reference or directly on the class name for `static` methods.

```java
Calculator calc = new Calculator(); // Create an object
int result = calc.add(5, 3);        // Calls 'add' method on the 'calc' object
System.out.println(result);         // Output: 8

calc.showCalculationResult(10, 20); // Calls 'showCalculationResult'
// Output: Message: The sum of 10 and 20 is: 30
```

### Method Overloading: Flexibility with the Same Name
Method overloading allows a class to have multiple methods with the **same name** but **different parameter lists**. This provides flexibility and improves readability by allowing related operations to share a common name.

*   **Distinguishing Factors:** For methods to be overloaded, they must differ in one or more of the following:
    *   **Number of parameters:** `add(int, int)` vs `add(int, int, int)`.
    *   **Type of parameters:** `add(int, int)` vs `add(double, double)`.
    *   **Order of parameters:** `print(String, int)` vs `print(int, String)`.
*   **Return type alone is NOT sufficient:** You cannot overload a method purely by changing its return type.
*   **Resolution:** The Java compiler determines which overloaded method to call based on the arguments provided at **compile time** (this is Compile-time Polymorphism).

```java
public class Printer {
    public void print(int num) {
        System.out.println("Printing integer: " + num);
    }

    public void print(String text) {
        System.out.println("Printing string: " + text);
    }

    public void print(String text, int num) {
        System.out.println("Printing formatted: " + text + " " + num);
    }

    // public String print(int num) { return "Can't overload by return type!"; } // Compile-time error

    public static void main(String[] args) {
        Printer p = new Printer();
        p.print(10);        // Calls print(int)
        p.print("Hello");   // Calls print(String)
        p.print("Value:", 5); // Calls print(String, int)
    }
}
```

### `varargs` (Variable-length Arguments): Flexible Parameter Lists
Introduced in Java 5, `varargs` allow a method to accept zero or more arguments of a specified type.

*   **Syntax:** An ellipsis (`...`) after the parameter's type (e.g., `int... numbers`).
*   **Behavior:** Inside the method, the `varargs` parameter is treated as an array of the specified type.
*   **Rule:** A method can have only one `varargs` parameter, and it must be the **last** parameter in the method's parameter list.

```java
public class DynamicCalculator {
    public int sum(int... numbers) { // Accepts 0 or more int arguments
        int total = 0;
        for (int num : numbers) { // 'numbers' is treated as an int[]
            total += num;
        }
        return total;
    }

    public static void main(String[] args) {
        DynamicCalculator dc = new DynamicCalculator();
        System.out.println(dc.sum(1, 2, 3));         // Output: 6
        System.out.println(dc.sum(10, 20, 30, 40));  // Output: 100
        System.out.println(dc.sum());                // Output: 0 (empty array)
    }
}
```

## 2. Encapsulation: Protecting and Controlling Data

Encapsulation is one of the four fundamental principles of OOP (along with Inheritance, Polymorphism, and Abstraction). It is the mechanism of **bundling the data (attributes/fields) and the methods (behaviors) that operate on that data into a single unit (a class)**. Crucially, it also involves **restricting direct access** to some of an object's components (data), making the internal state private and exposing well-defined public interfaces.

### Benefits of Encapsulation
*   **Data Hiding (Information Hiding):** Protects an object's internal state from unauthorized or unintended external access and modification. This prevents misuse and maintains data integrity.
*   **Flexibility and Maintainability:** Allows internal implementation details of a class to be changed without affecting external code that uses the class. If a field's underlying data structure changes, only the class's methods need updating, not every piece of code that uses the field.
*   **Control:** Provides control over how data is accessed and modified. `setters` can include validation logic (e.g., an age cannot be negative), ensuring invariants are maintained.
*   **Modularity:** Classes become self-contained and easier to understand, test, and debug.

### Access Modifiers: The Enforcers of Encapsulation
Access modifiers (`public`, `private`, `protected`, default/package-private) are keywords that set the accessibility (visibility) of classes, fields, methods, and constructors.

| Modifier   | Within Class | Within Package | Within Subclass (same package) | Within Subclass (different package) | Anywhere (different package, non-subclass) |
| :--------- | :----------- | :------------- | :----------------------------- | :---------------------------------- | :----------------------------------------- |
| `private`  | Yes          | No             | No                             | No                                  | No                                         |
| `default`  | Yes          | Yes            | No                             | No                                  | No                                         |
| `protected`| Yes          | Yes            | Yes                            | Yes                                 | No                                         |
| `public`   | Yes          | Yes            | Yes                            | Yes                                 | Yes                                        |

1.  **`private`:** The most restrictive. A member declared `private` is accessible only from **within the same class**. This is the primary modifier for achieving data hiding and the essence of encapsulation.
2.  **`default` (Package-Private):** If no access modifier is specified, the member is accessible only from **within the same package**.
3.  **`protected`:** A member declared `protected` is accessible within the same package and also by **subclasses** (even if they are in a different package).
4.  **`public`:** The least restrictive. A member declared `public` is accessible from **anywhere**.

### `getters` and `setters` (Accessors and Mutators): The Public Interface
To allow controlled access to `private` fields, you typically provide `public` methods:
*   **`getters` (Accessor Methods):** Public methods that return the value of a private field (e.g., `getName()`).
*   **`setters` (Mutator Methods):** Public methods that set the value of a private field. These are where you can implement validation logic.

```java
public class Student {
    private String name; // Private fields - hidden from direct external access
    private int age;     

    public Student(String name, int age) { // Constructor uses setters for validation
        this.name = name;
        setAge(age); // Use setter for validation logic
    }

    // Public Getter for name
    public String getName() {
        return name;
    }

    // Public Setter for name
    public void setName(String name) {
        this.name = name;
    }

    // Public Getter for age
    public int getAge() {
        return age;
    }

    // Public Setter for age (with validation logic)
    public void setAge(int age) {
        if (age > 0 && age < 120) {
            this.age = age;
        } else {
            // Log an error, throw an exception, or set a default value
            throw new IllegalArgumentException("Age must be between 1 and 119.");
        }
    }

    public static void main(String[] args) {
        Student s = new Student("Alice", 20);
        System.out.println(s.getName()); // Output: Alice

        try {
            s.setAge(150); // This will throw an IllegalArgumentException
        } catch (IllegalArgumentException e) {
            System.err.println("Error: " + e.getMessage());
        }
        System.out.println("Student's age: " + s.getAge()); // Output: 20 (age not changed due to error)
    }
}
```
By making `name` and `age` `private`, we enforce that all interactions with these data points go through the controlled interface of `public` getter and setter methods. This is encapsulation in action, ensuring that an object's internal state remains valid.

---

### Links to Topics:
*   [Classes & Objects](01_classes_and_objects.md)
*   [Methods & Encapsulation](02_methods_and_encapsulation.md)
*   [Inheritance](03_inheritance.md)
*   [Polymorphism](04_polymorphism.md)
*   [Abstraction (Interfaces & Abstract Classes)](05_abstraction_interfaces_and_abstract_classes.md)
*   [Static & Final Keywords](06_static_and_final_keywords.md)
