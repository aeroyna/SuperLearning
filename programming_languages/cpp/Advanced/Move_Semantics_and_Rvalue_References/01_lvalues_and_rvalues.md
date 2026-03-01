# Lvalues and Rvalues

To understand move semantics, you first need to understand the difference between lvalues and rvalues.

In C++, every expression has a type and belongs to a **value category**. The two most important value categories are lvalue and rvalue.

## Lvalues

An lvalue (short for "left value") is an expression that refers to a memory location. Lvalues are things that you can take the address of. They can appear on the left-hand side of an assignment.

### Examples of lvalues:

*   Variables: `int x = 10;` (`x` is an lvalue)
*   Array elements: `arr[0]`
*   The result of dereferencing a pointer: `*ptr`
*   A function call that returns a reference: `get_value()`

A simple rule of thumb: **if you can take its address, it's an lvalue.**

```cpp
int x;
int* p = &x; // valid, x is an lvalue
```

## Rvalues

An rvalue (short for "right value") is an expression that is temporary and does not have a persistent memory location. Rvalues typically appear on the right-hand side of an assignment.

### Examples of rvalues:

*   Literals: `10`, `3.14`, `"hello"`
*   The result of an arithmetic expression: `x + y`
*   A function call that returns by value: `get_number()`

You cannot take the address of an rvalue.

```cpp
int* p = &10; // invalid, 10 is an rvalue
```

## The "Temporary" Nature of Rvalues

Rvalues represent temporary objects that are about to be destroyed. For example, in the expression `x = a + b`, the result of `a + b` is stored in a temporary, intermediate object. This temporary object is an rvalue.

Move semantics is a way to take advantage of the temporary nature of rvalues. Instead of copying the data from a temporary object, we can "steal" its resources, because we know that the temporary object is about to be destroyed anyway. This is the key idea behind move semantics.
