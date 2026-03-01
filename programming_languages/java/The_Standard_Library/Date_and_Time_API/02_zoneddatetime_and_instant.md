# ZonedDateTime and Instant

While `LocalDateTime` is useful for calendar math (e.g., "Christmas is on Dec 25"), it doesn't represent a specific moment on the timeline without a time zone. For absolute time, we use `Instant` and `ZonedDateTime`.

## 1. `Instant` (Machine Time)
Represents a specific point on the timeline, defined as the count of nanoseconds since the Unix Epoch (January 1, 1970, 00:00:00 UTC).

*   **Use case:** Timestamps for logs, database entries, calculating duration between events.
*   **Timezone:** Always UTC.

```java
import java.time.Instant;

Instant now = Instant.now(); 
System.out.println(now); // e.g., 2023-10-05T10:00:00.123Z (Z = UTC)

long epochSeconds = now.getEpochSecond(); // Unix timestamp
```

## 2. `ZonedDateTime` (Human Time with Zone)
Represents a date and time with a specific time zone (e.g., "Europe/Paris"). It handles Daylight Saving Time (DST) transitions automatically.

### 2.1 ZoneId
Time zones are represented by `ZoneId`.
```java
import java.time.ZoneId;
import java.time.ZonedDateTime;

ZoneId zone = ZoneId.of("America/New_York");
ZonedDateTime zdt = ZonedDateTime.now(zone);

System.out.println(zdt); 
// 2023-10-05T06:00:00.123-04:00[America/New_York]
```

### 2.2 Converting between Local and Zoned
```java
LocalDateTime ldt = LocalDateTime.now();
// Attach a zone to a local date-time
ZonedDateTime parisTime = ldt.atZone(ZoneId.of("Europe/Paris"));

// Convert to a different zone (same instant, different wall-clock time)
ZonedDateTime tokyoTime = parisTime.withZoneSameInstant(ZoneId.of("Asia/Tokyo"));
```

## 3. `OffsetDateTime`
Similar to `ZonedDateTime` but only contains the offset from UTC (e.g., +02:00) without the specific zone rules (e.g., "Europe/Paris"). Useful for network protocols or databases that store offsets but not full zone IDs.