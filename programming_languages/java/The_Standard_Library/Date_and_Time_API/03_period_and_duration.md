# Period and Duration

Java 8 separates the concept of time elapsed into two classes: `Period` (date-based) and `Duration` (time-based).

## 1. `Duration` (Time-based)
Measures an amount of time in seconds and nanoseconds. Used with `LocalTime`, `LocalDateTime`, or `Instant`.

```java
import java.time.Duration;
import java.time.LocalTime;

LocalTime start = LocalTime.of(10, 0);
LocalTime end = LocalTime.of(10, 30);

Duration d = Duration.between(start, end);
System.out.println(d.toMinutes()); // 30

Duration twoHours = Duration.ofHours(2);
```

## 2. `Period` (Date-based)
Measures an amount of time in years, months, and days. Used with `LocalDate`.

```java
import java.time.LocalDate;
import java.time.Period;

LocalDate bday = LocalDate.of(1990, 1, 1);
LocalDate today = LocalDate.now();

Period p = Period.between(bday, today);
System.out.println("You are " + p.getYears() + " years old.");

Period tenDays = Period.ofDays(10);
LocalDate future = today.plus(tenDays);
```

## 3. `ChronoUnit`
A generic way to calculate time differences using the `java.time.temporal.ChronoUnit` enum.

```java
import java.time.temporal.ChronoUnit;

long daysBetween = ChronoUnit.DAYS.between(bday, today); // Total days
long minutesBetween = ChronoUnit.MINUTES.between(start, end);
```