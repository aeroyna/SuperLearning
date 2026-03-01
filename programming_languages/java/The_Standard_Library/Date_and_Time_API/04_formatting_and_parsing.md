# Formatting and Parsing

Parsing (String -> Date) and Formatting (Date -> String) are handled by the `DateTimeFormatter` class. Unlike `SimpleDateFormat`, `DateTimeFormatter` is **thread-safe**.

## 1. Standard Formatters
The class provides many pre-defined ISO formatters.

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

LocalDateTime now = LocalDateTime.now();

System.out.println(now.format(DateTimeFormatter.ISO_DATE)); // 2023-10-05
System.out.println(now.format(DateTimeFormatter.ISO_DATE_TIME)); // 2023-10-05T10:00:00.123
```

## 2. Custom Patterns
You can define custom patterns using letters (y=year, M=month, d=day, H=hour, m=minute, s=second).

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");

// Formatting
String formatted = now.format(formatter);
System.out.println(formatted); // 05/10/2023 10:00:00

// Parsing
String input = "05/10/2023 10:00:00";
LocalDateTime parsed = LocalDateTime.parse(input, formatter);
```

## 3. Localization
To format dates with localized names (e.g., "5 Octobre"), pass a `Locale`.

```java
import java.util.Locale;

DateTimeFormatter frenchFmt = DateTimeFormatter.ofPattern("d MMMM yyyy", Locale.FRENCH);
System.out.println(now.format(frenchFmt)); // 5 octobre 2023
```