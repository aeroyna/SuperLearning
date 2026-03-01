# Exception Safety

Exception safety is a property of a piece of code that describes how it behaves in the presence of exceptions. There are several levels of exception safety:

1.  **No-throw Guarantee:** The function will not throw any exceptions. This is the strongest guarantee. You can declare a function as `noexcept` to indicate this.

2.  **Strong Exception Safety:** If an exception is thrown, the state of the program is rolled back to the state it was in before the function was called. This is also known as "commit or rollback" semantics.

3.  **Basic Exception Safety:** If an exception is thrown, the program is in a valid but unspecified state. No resources are leaked, and all invariants are preserved. (An invariant is a condition that is always true for a class or data structure).

4.  **No Exception Safety:** If an exception is thrown, the program may be in an invalid state, and resources may be leaked. This is the weakest guarantee and should be avoided.

## Achieving Exception Safety

### RAII

As we saw in the "Memory Management" section, the RAII idiom is the most important tool for achieving exception safety. By wrapping resources in objects, you can ensure that they are properly released when an exception is thrown.

### Copy-and-Swap Idiom

The copy-and-swap idiom is a technique used to provide strong exception safety for class assignment operators.

```cpp
class MyClass {
    // ...
public:
    MyClass& operator=(MyClass other) { // pass by value to create a copy
        swap(*this, other);
        return *this;
    }

    friend void swap(MyClass& first, MyClass& second) noexcept {
        // swap the contents of first and second
        using std::swap;
        swap(first.data, second.data);
    }
private:
    SomeData* data;
};
```
How it works:
1.  The `other` object is created as a copy of the object on the right-hand side of the assignment. This can throw an exception (e.g., if memory allocation fails). If it does, the original object (`*this`) is unaffected.
2.  The `swap` function is called to swap the data of the original object and the copy. The `swap` function should be `noexcept`, meaning it is guaranteed not to throw.
3.  When the function returns, `other` goes out of scope, and its destructor frees the old data of the original object.

This ensures that the assignment is an "all or nothing" operation.

## `noexcept` Specifier

The `noexcept` specifier can be used to declare that a function will not throw any exceptions.

```cpp
void my_function() noexcept;
```
If a function declared `noexcept` does throw an exception, `std::terminate` is called, and the program is terminated.

The `noexcept` specifier is an important optimization tool. The compiler can generate more efficient code for `noexcept` functions, especially for move constructors and move assignment operators. For example, `std::vector` will use move semantics when it needs to resize, but only if the move constructor of the element type is `noexcept`.
