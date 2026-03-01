# References

A reference is an alias for an existing variable. It is like a constant pointer that is automatically dereferenced.

## Declaring References

You can declare a reference by using the `&` symbol.

```cpp
type& reference_name = existing_variable;
```

### Example

```cpp
int x = 10;
int& ref = x; // ref is a reference to x
```

## Properties of References

*   **Must be initialized:** A reference must be initialized when it is declared.
*   **Cannot be changed:** A reference cannot be changed to refer to another variable.
*   **No null references:** There are no null references.

### Example

```cpp
#include <iostream>

int main() {
    int x = 10;
    int& ref = x;

    std::cout << "x = " << x << std::endl;
    std::cout << "ref = " << ref << std::endl;

    ref = 20; // changing ref also changes x

    std::cout << "x = " << x << std::endl;
    std::cout << "ref = " << ref << std::endl;

    return 0;
}
```

## References vs. Pointers

| References                                    | Pointers                                      |
|-----------------------------------------------|-----------------------------------------------|
| Must be initialized when declared.            | Can be uninitialized.                         |
| Cannot be null.                               | Can be null (`nullptr`).                      |
| Cannot be changed to refer to another variable. | Can be changed to point to another variable.  |
| Use the dot (`.`) operator to access members. | Use the arrow (`->`) operator to access members. |
| Generally safer and easier to use.            | More flexible, but also more error-prone.     |

### Example: Pass by Reference vs. Pass by Pointer

We have already seen how to use references and pointers to pass arguments to functions.

**Pass by Reference:**
```cpp
void increment(int& n) {
    n++;
}

int main() {
    int x = 5;
    increment(x); // x is now 6
}
```

**Pass by Pointer:**
```cpp
void increment(int* n) {
    (*n)++;
}

int main() {
    int x = 5;
    increment(&x); // x is now 6
}
```

Passing by reference is often preferred because the syntax is cleaner and it avoids the possibility of a null pointer.
