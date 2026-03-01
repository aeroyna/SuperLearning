# Date and Time API

Handling dates and times is a complex task due to leap years, time zones, and daylight saving time changes. Java 8 introduced the `java.time` package (JSR-310) to provide a comprehensive, consistent, and thread-safe model for date and time manipulation.

## In this chapter, you will learn:
*   [**LocalDate, LocalTime, LocalDateTime**](01_localdate_localtime_localdatetime.md): Working with dates and times without time zones (human timeline).
*   [**ZonedDateTime and Instant**](02_zoneddatetime_and_instant.md): Working with absolute machine time and specific time zones.
*   [**Period and Duration**](03_period_and_duration.md): Calculating the amount of time between two points (date-based vs. time-based).
*   [**Formatting and Parsing**](04_formatting_and_parsing.md): Converting dates to strings and vice-versa using `DateTimeFormatter`.