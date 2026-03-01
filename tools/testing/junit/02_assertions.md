# Assertions

JUnit 5 assertions are static methods in the `org.junit.jupiter.api.Assertions` class.

## Common Assertions

*   `assertEquals(expected, actual)`
*   `assertNotEquals(unexpected, actual)`
*   `assertTrue(condition)`
*   `assertFalse(condition)`
*   `assertNull(object)`
*   `assertNotNull(object)`
*   `assertThrows(Exception.class, () -> { ... })`

## Grouped Assertions (`assertAll`)
In standard assertions, if the first fails, the second isn't executed. `assertAll` executes **all** of them and reports multiple failures at once.

```java
assertAll("person",
    () -> assertEquals("John", person.getFirstName()),
    () -> assertEquals("Doe", person.getLastName())
);
```
