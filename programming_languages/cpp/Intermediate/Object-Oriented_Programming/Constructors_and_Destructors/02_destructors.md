# Destructors

A destructor is a special member function that is automatically called when an object of a class goes out of scope or is explicitly destroyed by a call to `delete`.

## Properties of Destructors

*   A destructor has the same name as the class, preceded by a tilde (`~`).
*   A destructor does not have a return type and does not take any arguments.
*   There can only be one destructor in a class.

## When are destructors called?

A destructor is called when:
1.  A normal (local) object goes out of scope.
2.  The `delete` operator is called on a pointer to an object.
3.  A temporary object is destroyed.

### Example

```cpp
#include <iostream>

class MyClass {
public:
    MyClass() {
        std::cout << "Constructor called!" << std::endl;
    }

    ~MyClass() {
        std::cout << "Destructor called!" << std::endl;
    }
};

void create_object() {
    MyClass obj; // obj is created here
} // obj goes out of scope and is destroyed here

int main() {
    create_object();

    MyClass* ptr = new MyClass();
    delete ptr; // explicitly destroy the object

    return 0;
}
```

Output:
```
Constructor called!
Destructor called!
Constructor called!
Destructor called!
```

## Purpose of Destructors

The main purpose of a destructor is to free up resources that the object may have acquired during its lifetime. This is especially important when an object allocates memory dynamically.

### Example: Destructor for a class with dynamic memory

```cpp
#include <iostream>

class MyArray {
private:
    int* data;
    int size;

public:
    MyArray(int s) : size(s) {
        data = new int[size];
        std::cout << "Array of size " << size << " created." << std::endl;
    }

    ~MyArray() {
        delete[] data;
        std::cout << "Array destroyed." << std::endl;
    }
};

int main() {
    MyArray arr(10);
    return 0; // arr is destroyed here
}
```

In this example, the destructor ensures that the memory allocated for the array is deallocated when the `MyArray` object is destroyed, preventing a memory leak.
