# Regular Expressions

Regular expressions (regex or regexp) are powerful tools for pattern matching within text. Java provides robust support for regular expressions through the `java.util.regex` package, primarily with the `Pattern` and `Matcher` classes.

## 1. Basic Concepts

*   **Pattern:** A sequence of characters that defines a search pattern.
*   **Matcher:** An engine that performs match operations on a `CharSequence` (like `String`) by interpreting a `Pattern`.

## 2. Key Classes in `java.util.regex`

### `Pattern` Class
*   **Purpose:** Represents a compiled regular expression.
*   **Creation:** Use `Pattern.compile(String regex)` to compile a regex string into a `Pattern` object. This is an expensive operation, so compile once and reuse.

### `Matcher` Class
*   **Purpose:** Interprets a `Pattern` and performs match operations against an input string.
*   **Creation:** Use `pattern.matcher(CharSequence input)` to get a `Matcher` object.

## 3. Basic Pattern Matching

### Example
```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class BasicRegex {
    public static void main(String[] args) {
        String text = "Java is fun. Java is powerful.";
        String regex = "Java"; // The pattern we are looking for

        Pattern pattern = Pattern.compile(regex); // Compile the regex
        Matcher matcher = pattern.matcher(text); // Create a Matcher object

        int count = 0;
        // find() attempts to find the next subsequence of the input sequence that matches the pattern.
        while (matcher.find()) {
            count++;
            System.out.println("Found match " + count + " at index " + matcher.start() + " to " + matcher.end());
            // matcher.start() returns the starting index of the match
            // matcher.end() returns the offset after the last character matched
        }
        // Output:
        // Found match 1 at index 0 to 4
        // Found match 2 at index 13 to 17
    }
}
```

## 4. Common Regex Metacharacters and Quantifiers

### Metacharacters (Special Characters)
*   `.`: Any character (except newline).
*   `\d`: A digit (`[0-9]`).
*   `\D`: A non-digit (`[^0-9]`).
*   `\s`: A whitespace character (`[ 	
]`).
*   `\S`: A non-whitespace character.
*   `\w`: A word character (`[a-zA-Z0-9_]`).
*   `\W`: A non-word character.
*   `[abc]`: A single character from the set `a`, `b`, or `c`.
*   `[^abc]`: A single character NOT from the set `a`, `b`, or `c`.
*   `[a-zA-Z]`: A single character from 'a' to 'z' or 'A' to 'Z'.
*   `^`: Start of a line.
*   `$`: End of a line.
*   `\b`: Word boundary.
*   `\B`: Non-word boundary.
*   `|`: OR operator (e.g., `cat|dog`).
*   `()`: Grouping.

### Quantifiers (How many times a character/group appears)
*   `?`: Once or not at all (0 or 1).
*   `*`: Zero or more times.
*   `+`: One or more times.
*   `{n}`: Exactly `n` times.
*   `{n,}`: At least `n` times.
*   `{n,m}`: At least `n` but not more than `m` times.

### Example with Quantifiers
```java
String phoneRegex = "\\d{3}-\\d{3}-\\d{4}"; // Matches XXX-XXX-XXXX
String phoneNumber = "123-456-7890";
System.out.println(Pattern.matches(phoneRegex, phoneNumber)); // Output: true
```
*   **Note the double backslash `\\`:** In Java strings, a single backslash `\` is an escape character. To represent a literal backslash in a regex, you need to escape it: `\\d` for `\d`.

## 5. Capturing Groups

Parentheses `()` not only group parts of a regex but also "capture" the matched text. You can then retrieve these captured groups.

### Example
```java
String dateRegex = "(\\d{4})-(\\d{2})-(\\d{2})"; // Captures year, month, day
String dateString = "Today's date is 2023-10-26.";

Pattern pattern = Pattern.compile(dateRegex);
Matcher matcher = pattern.matcher(dateString);

if (matcher.find()) { // Finds the first match
    System.out.println("Full Match: " + matcher.group(0)); // Group 0 is the entire match
    System.out.println("Year: " + matcher.group(1));    // First captured group
    System.out.println("Month: " + matcher.group(2));   // Second captured group
    System.out.println("Day: " + matcher.group(3));     // Third captured group
}
// Output:
// Full Match: 2023-10-26
// Year: 2023
// Month: 10
// Day: 26
```

## 6. Replacing Text

The `Matcher` class provides methods to replace matched substrings.

### `replaceAll()` and `replaceFirst()`
```java
String text = "The cat sat on the mat. The cat is happy.";
Pattern pattern = Pattern.compile("cat");
Matcher matcher = pattern.matcher(text);

String newTextAll = matcher.replaceAll("dog");
System.out.println(newTextAll); // Output: The dog sat on the mat. The dog is happy.

String newTextFirst = matcher.replaceFirst("dog"); // Note: Need a new matcher for replaceFirst after find()
System.out.println(new Matcher(Pattern.compile("cat"), text).replaceFirst("dog")); // Better to reset matcher or create new
// Output: The dog sat on the mat. The cat is happy.
```

## 7. `String` Class Regex Methods

For simple pattern matching and replacement, the `String` class provides convenient methods that internally use `Pattern` and `Matcher`.

*   `String.matches(String regex)`: Checks if the entire string matches the given regex.
*   `String.split(String regex)`: Splits the string by the given regex delimiter.
*   `String.replaceAll(String regex, String replacement)`: Replaces all occurrences.
*   `String.replaceFirst(String regex, String replacement)`: Replaces the first occurrence.

```java
String email = "test@example.com";
System.out.println(email.matches("\\w+@\\w+\\.\\w+\e")); // true

String sentence = "apple,banana,orange";
String[] fruits = sentence.split(","); // ["apple", "banana", "orange"]
```

## 8. Regex Flags

You can specify flags to modify the matching behavior.
*   `Pattern.CASE_INSENSITIVE`
*   `Pattern.MULTILINE`
*   `Pattern.DOTALL` (makes `.` match all characters, including newlines)

```java
Pattern pattern = Pattern.compile("java", Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher("Java is powerful.");
System.out.println(matcher.find()); // true
```

Regular expressions are a vast topic, and this provides a foundation. Mastery comes with practice and understanding the specific requirements of the text you are trying to match or manipulate.
