# Mockito Basics

Mockito is a mocking framework for unit tests. It allows you to create dummy objects (mocks) to simulate the behavior of dependencies.

## Creating Mocks

```java
// Option 1: Static method
List mockedList = mock(List.class);

// Option 2: Annotation (requires initialization)
@Mock
List mockedList;
```

## Stubbing (`when`...`thenReturn`)
Define how the mock should behave.

```java
// When get(0) is called, return "first"
when(mockedList.get(0)).thenReturn("first");

// When get(1) is called, throw an exception
when(mockedList.get(1)).thenThrow(new RuntimeException());
```

## Argument Matchers
Use `any()` when the specific argument doesn't matter.

```java
when(mockedList.get(anyInt())).thenReturn("element");
```
