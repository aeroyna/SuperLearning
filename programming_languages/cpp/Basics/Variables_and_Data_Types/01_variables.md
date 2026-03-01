# Variables in C++

A variable is a name given to a memory location. It is the basic unit of storage in a program. The value stored in a variable can be changed during program execution.

## Declaring Variables

In C++, all variables must be declared before they are used. A typical variable declaration is:

```cpp
type variable_name;
```

For example:

```cpp
int age;
double salary;
char initial;
```

You can also declare multiple variables of the same type in a single line:

```cpp
int x, y, z;
```

## Initializing Variables

You can initialize a variable at the time of declaration:

```cpp
int age = 30;
double salary = 50000.0;
char initial = 'J';
```

C++ also supports other forms of initialization:

*   **Constructor Initialization:**

    ```cpp
    int age(30);
    ```

*   **Uniform Initialization (C++11 and later):**

    ```cpp
    int age{30};
    ```

    Uniform initialization is generally preferred because it is more consistent and can prevent some types of errors (e.g., narrowing conversions).

## Variable Scope

The scope of a variable is the part of the program where the variable can be accessed. In C++, there are two main types of scope:

*   **Local Scope:** A variable declared inside a function or a block is a local variable. It can only be used inside that function or block.

    ```cpp
    void myFunction() {
        int localVar = 10; // local variable
        // localVar can be used here
    }

    // localVar cannot be used here
    ```

*   **Global Scope:** A variable declared outside of all functions is a global variable. It can be accessed from any part of the program.

    ```cpp
    int globalVar = 20; // global variable

    void myFunction() {
        // globalVar can be used here
    }

    int main() {
        // globalVar can be used here
        return 0;
    }
    ```

    It is generally a good practice to minimize the use of global variables as they can make the program harder to read and maintain.
