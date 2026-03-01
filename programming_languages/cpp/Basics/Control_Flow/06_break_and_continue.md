# `break` and `continue`

`break` and `continue` are two statements that alter the normal flow of a loop.

## `break` Statement

The `break` statement is used to terminate a loop (`for`, `while`, `do-while`) or a `switch` statement.

### Example with `for` loop

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 10; ++i) {
        if (i == 5) {
            break; // exit the loop when i is 5
        }
        std::cout << "i = " << i << std::endl;
    }
    // The loop terminates here
    std::cout << "Loop finished." << std::endl;

    return 0;
}
```

Output:
```
i = 0
i = 1
i = 2
i = 3
i = 4
Loop finished.
```

## `continue` Statement

The `continue` statement is used to skip the current iteration of a loop and move to the next iteration.

### Example with `for` loop

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 5; ++i) {
        if (i == 2) {
            continue; // skip the rest of the loop when i is 2
        }
        std::cout << "i = " << i << std::endl;
    }

    return 0;
}
```

Output:
```
i = 0
i = 1
i = 3
i = 4
```
As you can see, the value `i = 2` was not printed because the `continue` statement skipped that iteration.
