# Introduction to C++

Welcome to the C++ course! This course is designed to take you from the basics to advanced topics in C++.

## What is C++?

C++ is a general-purpose programming language created by Bjarne Stroustrup as an extension of the C programming language, or "C with Classes". It has imperative, object-oriented and generic programming features. C++ is a powerful, high-performance language used in a wide range of applications, including:

*   **Operating Systems:** Windows, macOS, and Linux have components written in C++.
*   **Game Development:** Major game engines like Unreal Engine and Unity are built with C++.
*   **Web Browsers:** Google Chrome and Mozilla Firefox use C++ extensively.
*   **Embedded Systems:** C++ is used in real-time systems where performance is critical.
*   **High-Frequency Trading:** C++ is the language of choice for financial applications that require low latency.

## "Hello, World!" in C++

Here is a simple "Hello, World!" program in C++:

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

### Explanation:

*   `#include <iostream>`: This line includes the input/output stream library, which allows you to print text to the console.
*   `int main()`: This is the main function where the program execution begins.
*   `std::cout << "Hello, World!" << std::endl;`: This line prints "Hello, World!" to the console.
    *   `std::cout` is the standard output stream.
    *   `<<` is the insertion operator.
    *   `std::endl` inserts a newline character and flushes the stream.
*   `return 0;`: This line indicates that the program has executed successfully.

## How to Compile and Run

To compile and run this program, you will need a C++ compiler like g++.

1.  Save the code in a file named `hello.cpp`.
2.  Open a terminal and compile the code using the following command:

    ```bash
    g++ hello.cpp -o hello
    ```

3.  Run the compiled program:

    ```bash
    ./hello
    ```

    You should see the output:

    ```
    Hello, World!
    ```

This is just a brief introduction. In the next sections, we will dive deeper into the fundamentals of C++.
