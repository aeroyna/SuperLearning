# Methods and Encapsulation

Methods define the **behaviors** that objects of a class can perform. Encapsulation is a core OOP principle that bundles data and the methods that operate on the data into a single unit (the class), and restricts direct access to some of the object's components.

## 1. Methods

### Method Declaration
```java
// Access_Modifier Return_Type Method_Name(Parameters) {
//     // Method body
//     return value; // if Return_Type is not void
// }

public class Calculator {
    // Method that adds two integers and returns their sum
    public int add(int a, int b) {
        int sum = a + b;
        return sum; // Returns an int value
    }

    // Method that prints a message and returns nothing (void)
    public void displayMessage(String message) {
        System.out.println("Message: " + message);
    }
}
```
*   **`Access_Modifier`:** (e.g., `public`, `private`, `protected`, default). Controls visibility.
*   **`Return_Type`:** The type of data the method will return. If no value is returned, use `void`.
*   **`Method_Name`:** Follows `camelCase` convention (e.g., `calculateTotal`).
*   **`Parameters`:** A comma-separated list of `type name` pairs. Methods can have zero or more parameters.
*   **`Method Body`:** The code that defines what the method does.

### Calling Methods
```java
Calculator calc = new Calculator();
int result = calc.add(5, 3); // result will be 8
calc.displayMessage("Calculation complete.");
```

## 2. Java is Always Pass-by-Value

One of the most common points of confusion is whether Java passes parameters by value or by reference. **Java is strictly Pass-by-Value.**

*   **For Primitives:** The actual value (e.g., `5`, `true`) is copied and passed to the method. Changes to the parameter inside the method do not affect the original variable.
*   **For Objects:** The **value of the reference** (the memory address) is copied and passed.
    *   This means the method receives a copy of the pointer to the *same* object.
    *   **Consequence:** You *can* modify the object's state (e.g., `person.setName("Bob")`) and the caller will see the change.
    *   **Crucial Distinction:** You *cannot* reassign the original reference variable itself. If you do `person = new Person("New")` inside the method, only the local copy of the reference is changed. The caller's variable still points to the original object.

### Method Overloading
Allows a class to have multiple methods with the **same name** but **different parameter lists**.
*   **Distinguishing factor:** Number of parameters, type of parameters, or order of parameters.
*   **Return type alone is NOT enough** to distinguish overloaded methods.

```java
public class Printer {
    public void print(int num) {
        System.out.println("Printing int: " + num);
    }

    public void print(String text) {
        System.out.println("Printing string: " + text);
    }

    public void print(String text, int num) {
        System.out.println("Printing: " + text + " " + num);
    }
}

Printer p = new Printer();
p.print(10);        // Calls print(int)
p.print("Hello");   // Calls print(String)
p.print("Value:", 5); // Calls print(String, int)
```

### `varargs` (Variable-length Arguments)
Allows a method to accept zero or more arguments of a specified type.
*   Represented by an ellipsis (`...`) after the parameter's type.
*   Must be the **last** parameter in a method's parameter list.
*   Inside the method, it's treated as an array.

```java
public class Calculator {
    public int sum(int... numbers) { // Accepts 0 or more int arguments
        int total = 0;
        for (int num : numbers) {
            total += num;
        }
        return total;
    }
}

Calculator calc = new Calculator();
System.out.println(calc.sum(1, 2, 3));      // Output: 6
System.out.println(calc.sum(10, 20, 30, 40)); // Output: 100
System.out.println(calc.sum());             // Output: 0
```

---

## 2. Encapsulation

Encapsulation is one of the four fundamental principles of OOP. It's the mechanism of **bundling the data (attributes) and methods (behaviors) that operate on the data into a single unit (a class)**. It also involves **restricting direct access** to some of an object's components (data), which is achieved through **Access Modifiers**.

### Benefits of Encapsulation
*   **Data Hiding:** Protects an object's internal state from unauthorized access or modification.
*   **Flexibility:** Allows changes to the internal implementation of a class without affecting external code that uses the class.
*   **Maintainability:** Easier to debug and maintain code.
*   **Control:** Allows validation of data (e.g., age cannot be negative).

### Access Modifiers
Control the visibility/accessibility of classes, attributes, methods, and constructors.

| Modifier | Class | Package | Subclass | World |
| :------- | :---- | :------ | :------- | :---- |
| `public` | Y     | Y       | Y        | Y     |
| `protected` | Y   | Y       | Y        | N     |
| `default` (no modifier) | Y | Y | N | N |
| `private` | Y     | N       | N        | N     |

1.  **`public`:** The member is accessible from anywhere.
2.  **`private`:** The member is accessible only from **within the same class**. This is the primary modifier for achieving data hiding.
3.  **`protected`:** The member is accessible within the same package and also by subclasses (even if in a different package).
4.  **`default` (or Package-Private):** If no access modifier is specified, the member is accessible only from **within the same package**.

### `getters` and `setters` (Accessors and Mutators)
To access and modify `private` attributes, you provide `public` methods.
*   **`getters`:** Public methods that return the value of a private field (e.g., `getName()`).
*   **`setters`:** Public methods that set the value of a private field. They can include validation logic (e.g., `setAge(int age)`).

```java
public class Student {
    private String name; // private attribute
    private int age;     // private attribute

    public Student(String name, int age) {
        this.name = name;
        setAge(age); // Use setter for validation
    }

    // Getter for name
    public String getName() {
        return name;
    }

    // Setter for name
    public void setName(String name) {
        this.name = name;
    }

    // Getter for age
    public int getAge() {
        return age;
    }

    // Setter for age (with validation)
    public void setAge(int age) {
        if (age > 0 && age < 120) {
            this.age = age;
        } else {
            System.out.println("Invalid age provided.");
            // Or throw an IllegalArgumentException
        }
    }
}

// Usage
Student s = new Student("Alice", 20);
System.out.println(s.getName()); // Output: Alice
s.setAge(150); // Output: Invalid age provided.
System.out.println(s.getAge()); // Output: 20 (age not changed)
```

By making `name` and `age` `private`, we prevent direct manipulation from outside the class, enforcing that all interactions go through the `public` getter and setter methods. This is encapsulation in practice.