# Rvalue References

C++11 introduced a new kind of reference called an **rvalue reference**, which is declared using `&&`.

*   **Lvalue Reference (`&`):** Binds to an lvalue.
*   **Rvalue Reference (`&&`):** Binds to an rvalue.

```cpp
int x = 10;
int& ref1 = x;      // OK: ref1 is an lvalue reference bound to an lvalue
// int& ref2 = 10;  // Error: cannot bind an lvalue reference to an rvalue

int&& ref3 = 10;    // OK: ref3 is an rvalue reference bound to an rvalue
// int&& ref4 = x;  // Error: cannot bind an rvalue reference to an lvalue
```

The one exception to this rule is that a `const` lvalue reference can bind to an rvalue. This is an older feature of C++ that is used for passing arguments to functions efficiently without making a copy.

```cpp
const int& ref5 = 10; // OK
```

## Purpose of Rvalue References

The main purpose of rvalue references is to allow us to **overload functions based on the value category of the arguments**. This means we can have one version of a function that takes an lvalue reference, and another version that takes an rvalue reference.

```cpp
#include <iostream>

void my_function(int& x) {
    std::cout << "lvalue reference version" << std::endl;
}

void my_function(int&& x) {
    std::cout << "rvalue reference version" << std::endl;
}

int main() {
    int a = 10;
    my_function(a);      // calls the lvalue reference version
    my_function(20);     // calls the rvalue reference version
    my_function(a + 5);  // calls the rvalue reference version

    return 0;
}
```

This ability to differentiate between lvalues and rvalues is the foundation of move semantics. The rvalue reference version of a function can safely "steal" the resources of its argument, because it knows that the argument is a temporary object.

## `std::move`

`std::move` is a function in the `<utility>` header that casts its argument to an rvalue. It doesn't actually move anything; it just allows you to treat an lvalue as if it were an rvalue.

This is useful when you want to call the rvalue reference version of a function with an lvalue argument.

```cpp
#include <iostream>
#include <utility> // for std::move

void my_function(int&& x) {
    std::cout << "rvalue reference version" << std::endl;
}

int main() {
    int a = 10;
    // my_function(a); // Error
    my_function(std::move(a)); // OK, calls the rvalue reference version

    // After this point, 'a' is in a valid but unspecified state.
    // You should not use 'a' again without re-initializing it.

    return 0;
}
```

**Warning:** After you have used `std::move` on an object, you should assume that its resources have been stolen. Do not use the object again unless you re-initialize it.

In the next section, we will see how rvalue references and `std::move` are used to implement move constructors and move assignment operators.
