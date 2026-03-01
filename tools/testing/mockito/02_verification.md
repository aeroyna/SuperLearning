# Verification

Stubbing defines what a mock *returns*. Verification checks that a mock *was called*.

## Basic Verification

```java
// Verify add("one") was called exactly once
verify(mockedList).add("one");

// Verify clear() was never called
verify(mockedList, never()).clear();

// Verify add("twice") was called 2 times
verify(mockedList, times(2)).add("twice");
```

## Capturing Arguments
Sometimes you want to check *what* argument was passed to a method.

```java
ArgumentCaptor<Person> argument = ArgumentCaptor.forClass(Person.class);

verify(mockRepo).save(argument.capture());

assertEquals("John", argument.getValue().getName());
```
