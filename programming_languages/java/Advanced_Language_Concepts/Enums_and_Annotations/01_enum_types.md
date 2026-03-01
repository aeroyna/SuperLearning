# Enum Types

An enumeration (enum) is a special data type that enables a variable to be a set of predefined constants. Enums were introduced in **Java 5**. They provide a type-safe and robust alternative to using `int` constants for representing a fixed set of values.

## 1. The Problem with Integer Constants

Before enums, developers often used `public static final int` for sets of related constants:
```java
public class DayConstants {
    public static final int MONDAY = 1;
    public static final int TUESDAY = 2;
    public static final int WEDNESDAY = 3;
    // ...
}

// Problem: No type safety
int day = DayConstants.MONDAY;
int month = 5;
if (day == month) { // This comparison is syntactically valid but logically flawed
    System.out.println("This is a logical error!");
}
```
*   **No Type Safety:** Any `int` could be passed where a day was expected.
*   **No Readability:** `System.out.println(1)` doesn't convey meaning.
*   **No Iteration:** Cannot easily loop through all possible values.

## 2. Basic Enum Declaration

An `enum` is declared using the `enum` keyword.
```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

### Usage
```java
Day today = Day.WEDNESDAY;

// Type-safe comparison
if (today == Day.WEDNESDAY) {
    System.out.println("It's Wednesday!");
}

// Cannot compare with unrelated types
// if (today == 3) { } // Compile-time error

// Output enum value
System.out.println(today); // Output: WEDNESDAY

// Iterate through all enum values
for (Day day : Day.values()) { // values() is implicitly generated
    System.out.println(day);
}
```

## 3. Enums Provide Type Safety

Now, a variable declared as type `Day` can **only** hold one of the predefined `Day` constants. This prevents logical errors and improves code robustness.

## 4. `enum` is a Class

In Java, an `enum` is actually a special kind of class. Each enum constant is an instance of this `enum` class, implicitly `public static final`.
Because enums are classes, they can have:
*   Fields (instance variables)
*   Methods
*   Constructors
*   Implement interfaces

(These advanced features are covered in the next chapter).

## 5. Built-in `enum` Methods
Every enum type automatically inherits several useful methods from `java.lang.Enum`.

*   **`name()`:** Returns the name of this enum constant, exactly as declared.
    *   `Day.MONDAY.name()` returns `"MONDAY"`.
*   **`ordinal()`:** Returns the ordinal (position) of this enum constant in its enum declaration (where the first constant is assigned an ordinal of 0).
    *   `Day.MONDAY.ordinal()` returns `0`.
    *   **Caution:** Do not rely on `ordinal()` for storing enum values, as changing the order of constants will break serialized data.
*   **`valueOf(String name)`:** Returns the enum constant of the specified enum type with the specified name.
    *   `Day.valueOf("MONDAY")` returns `Day.MONDAY`.
    *   Throws `IllegalArgumentException` if no constant with the specified name is found.
*   **`values()`:** Returns an array containing all the enum constants of this type in the order they are declared. (This is a static method, `Day.values()`).

```java
Day currentDay = Day.TUESDAY;
System.out.println("Name: " + currentDay.name());    // TUESDAY
System.out.println("Ordinal: " + currentDay.ordinal()); // 1

Day parsedDay = Day.valueOf("FRIDAY");
System.out.println("Parsed: " + parsedDay); // FRIDAY
```

## 6. Using Enums in `switch` Statements
Enums are particularly useful in `switch` statements, making the code much more readable and ensuring type safety.
```java
Day today = Day.SUNDAY;
switch (today) {
    case SATURDAY:
    case SUNDAY:
        System.out.println("It's the weekend!");
        break;
    default:
        System.out.println("It's a weekday.");
}
```
*   Notice you don't use `Day.SATURDAY` in the `case` label, just `SATURDAY`.

---
Enums enhance readability, maintainability, and type safety, making them a preferred way to handle fixed sets of related constants in Java applications.