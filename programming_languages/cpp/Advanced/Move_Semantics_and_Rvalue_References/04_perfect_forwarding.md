# Perfect Forwarding

Perfect forwarding is a C++11 feature that allows you to write a function template that can take any arguments (lvalues, rvalues, const, etc.) and forward them to another function while preserving their original value category.

## The Problem

Consider a factory function that creates an object and passes some arguments to its constructor.

```cpp
#include <memory>

class MyClass {
public:
    MyClass(int&, double) { /* ... */ } // constructor takes an lvalue ref and a double
};

template <typename T, typename Arg1, typename Arg2>
std::unique_ptr<T> factory(Arg1 arg1, Arg2 arg2) {
    return std::make_unique<T>(arg1, arg2);
}

int main() {
    int x = 10;
    factory<MyClass>(x, 3.14); // Error!
}
```
This fails because `arg1` inside the `factory` function is an lvalue (all named variables are lvalues), but the `MyClass` constructor expects an lvalue reference. When `arg1` is passed to `make_unique`, the compiler can't bind the lvalue `arg1` to the `int&` parameter of the `MyClass` constructor.

We could try to fix this by using rvalue references, but that would also fail. Perfect forwarding is the solution to this problem.

## Forwarding References

The key to perfect forwarding is a special kind of reference called a **forwarding reference** (also known as a "universal reference").

A forwarding reference is an rvalue reference to a template parameter.

```cpp
template <typename T>
void my_function(T&& param); // param is a forwarding reference
```

Forwarding references have a special rule:
*   If you pass an lvalue to `my_function`, `T` is deduced as an lvalue reference, and `param` becomes an lvalue reference (due to a rule called "reference collapsing").
*   If you pass an rvalue to `my_function`, `T` is deduced as a non-reference type, and `param` becomes an rvalue reference.

This allows `param` to bind to both lvalues and rvalues.

## `std::forward`

Now that we have a reference that can bind to anything, we need a way to forward it to the target function while preserving its original value category. This is what `std::forward` is for.

`std::forward` is a conditional cast. It casts its argument to an rvalue reference only if the argument was originally an rvalue.

## The Solution with Perfect Forwarding

Here is the corrected `factory` function using perfect forwarding:

```cpp
#include <memory>
#include <utility> // for std::forward

class MyClass {
public:
    MyClass(int&, double) { /* ... */ }
};

template <typename T, typename... Args> // using variadic templates
std::unique_ptr<T> factory(Args&&... args) {
    return std::make_unique<T>(std::forward<Args>(args)...);
}

int main() {
    int x = 10;
    auto p = factory<MyClass>(x, 3.14); // OK
    return 0;
}
```

How it works:
1.  The `factory` function is now a variadic template that takes forwarding references (`Args&&...`).
2.  In the `main` function, `x` is an lvalue and `3.14` is an rvalue.
3.  Inside `factory`, the type of the `x` argument is deduced as `int&`, and the type of the `3.14` argument is deduced as `double`. The parameters `args` become `int&` and `double&&`.
4.  `std::forward` is used to forward the arguments to `make_unique`. `std::forward` will forward `x` as an lvalue and `3.14` as an rvalue, which is what the `MyClass` constructor expects.

Perfect forwarding is a complex but powerful feature that is essential for writing generic code in modern C++. It is used in many places in the standard library, such as `std::vector::emplace_back`, `std::make_unique`, and `std::make_shared`.
