# Function Declaration and Definition

A function is a block of code that performs a specific task. Functions are used to break down a large program into smaller, more manageable parts.

## Function Declaration

A function declaration tells the compiler about a function's name, return type, and parameters. A function declaration is also known as a **function prototype**.

### Syntax

```cpp
return_type function_name(parameter_list);
```

### Example

```cpp
int add(int a, int b); // function prototype
```

A function declaration is not required if the function is defined before it is used. However, it is a good practice to declare all functions at the beginning of the program, usually in a header file.

## Function Definition

A function definition provides the actual body of the function.

### Syntax

```cpp
return_type function_name(parameter_list) {
    // body of the function
}
```

### Example

```cpp
int add(int a, int b) {
    return a + b;
}
```

## Putting it all together

Here is a complete example showing how to declare and define a function.

```cpp
#include <iostream>

// Function declaration (prototype)
int add(int a, int b);

int main() {
    int x = 10;
    int y = 20;
    int sum = add(x, y); // function call

    std::cout << "The sum is: " << sum << std::endl;

    return 0;
}

// Function definition
int add(int a, int b) {
    return a + b;
}
```

In this example, we have declared the `add` function before `main` and defined it after `main`. This allows `main` to call `add` even though the definition of `add` appears later in the code.
