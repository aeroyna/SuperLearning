# LocalDate, LocalTime, LocalDateTime: Deep Dive

## 1. Design Philosophy (JSR-310)
Designed by Stephen Colebourne (Joda-Time author).
*   **Immutable:** All instances are final. Modification methods return copies.
*   **Thread-Safe:** Safe to use in static fields.
*   **Clarity:** Methods are explicit (`of`, `from`, `parse`, `format`).

## 2. Internal Storage
*   `LocalDate`: Stores `year` (int), `month` (short), `day` (short). No time zone.
*   `LocalDateTime`: Composed of `LocalDate` + `LocalTime`.

## 3. Leap Seconds
The ISO-8601 standard allows for leap seconds (61 seconds in a minute).
*   **Java's Time Scale:** Java uses a simplified view of time where every day has exactly 86,400 seconds.
*   **Handling:** `java.time` "smears" or ignores leap seconds to maintain monotonically increasing time, aligning with standard OS clock behavior (Unix Time).

## 4. Arithmetic
Methods like `plusDays(1)` handle calendar complexity:
*   Adding 1 month to "Jan 31" -> "Feb 28" (Last day of Feb).
*   Adding 1 year to "Feb 29, 2020" (Leap) -> "Feb 28, 2021" (Non-leap).
