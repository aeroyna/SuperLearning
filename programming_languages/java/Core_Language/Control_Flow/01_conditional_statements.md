# Conditional Statements

Conditional statements allow your program to make decisions and execute different blocks of code based on certain conditions.

## 1. The `if-else` Statement

The most fundamental conditional construct.

### Basic `if`
Executes code if a condition is `true`.
```java
int score = 75;
if (score >= 60) {
    System.out.println("Passed!");
}
```

### `if-else`
Executes one block if `true`, another if `false`.
```java
int score = 55;
if (score >= 60) {
    System.out.println("Passed!");
} else {
    System.out.println("Failed!");
}
```

### `if-else if-else` Ladder
For multiple conditions. Evaluates from top to bottom. The first `true` condition's block executes, and the rest are skipped.
```java
int grade = 85;
if (grade >= 90) {
    System.out.println("A");
} else if (grade >= 80) {
    System.out.println("B");
} else if (grade >= 70) {
    System.out.println("C");
} else {
    System.out.println("F");
}
```

### Nested `if`
An `if` statement inside another `if`.
```java
boolean isAdmin = true;
boolean isLoggedIn = true;

if (isLoggedIn) {
    if (isAdmin) {
        System.out.println("Welcome, Admin!");
    } else {
        System.out.println("Welcome, User!");
    }
} else {
    System.out.println("Please log in.");
}
```

### Ternary Operator (`? :`)
A concise way to write simple `if-else` statements, especially for assignments.
```java
int age = 20;
String status = (age >= 18) ? "Adult" : "Minor";
System.out.println(status); // Output: Adult
```

---

## 2. The `switch` Statement

Used for selecting one of many code blocks to execute based on the value of a variable.

### Traditional `switch` (Java 8 and earlier)
*   The `switch` variable can be `byte`, `short`, `char`, `int`, `String` (from Java 7), `enum`, or their wrapper classes.
*   `case` values must be literals or `final` constants.
*   The `break` statement is crucial to exit the `switch` block. Without it, execution "falls through" to the next `case`.
*   `default` is optional, executed if no `case` matches.

```java
int day = 3;
String dayName;
switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    default:
        dayName = "Invalid Day";
        break; // break even in default is good practice
}
System.out.println(dayName); // Output: Wednesday
```

### `switch` Expressions (Java 14+)
Introduced to address the "fall-through" problem and make `switch` more expressive. They can return a value.

#### `->` (Arrow) Syntax
*   No need for `break`; it implicitly breaks after the execution.
*   Can have multiple `case` labels separated by commas.

```java
int day = 3;
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4, 5 -> "Thursday or Friday"; // Multiple labels
    default -> "Invalid Day";
};
System.out.println(dayName); // Output: Wednesday
```

#### `yield` Keyword (for multi-line blocks)
If a `case` needs multiple statements, use a code block `{}` and `yield` the result.
```java
int day = 3;
String dayType = switch (day) {
    case 1, 2, 3, 4, 5 -> {
        System.out.println("It's a weekday!");
        yield "Weekday"; // yield returns the value
    }
    case 6, 7 -> {
        System.out.println("It's the weekend!");
        yield "Weekend";
    }
    default -> {
        System.out.println("Unknown day!");
        yield "Unknown";
    }
};
System.out.println(dayType); // Output: It's a weekday! 
 Weekday
```

### `switch` with `enum`s
`switch` statements work very well with `enum` types, often providing clearer and safer code.
```java
public enum TrafficLight { RED, YELLOW, GREEN }

public String getAction(TrafficLight light) {
    return switch (light) {
        case RED -> "Stop";
        case YELLOW -> "Prepare to stop";
        case GREEN -> "Go";
    };
}
```

---

## 3. Comparing `if-else` vs `switch`

| Feature           | `if-else` | `switch` (traditional) | `switch` (expression) |
| :---------------- | :-------- | :--------------------- | :-------------------- |
| **Flexibility**   | High      | Limited to single variable | Moderate              |
| **Conditions**    | Any `boolean` expression | Equality check only    | Equality check, `yield` for logic |
| **Fall-through**  | No        | Yes (with `break`)     | No (implicit `break`) |
| **Return Value**  | No        | No                     | Yes                   |
| **Readability**   | Good for few conditions | Good for many discrete values | Excellent for many discrete values |
| **Introduced in** | Java 1    | Java 1                 | Java 14               |

Choose `switch` when you have a single variable being compared against multiple discrete values. For complex boolean logic, `if-else` is more suitable.

```