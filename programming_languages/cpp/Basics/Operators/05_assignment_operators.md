# Assignment Operators

Assignment operators are used to assign values to variables.

## Simple Assignment Operator (`=`)

The simple assignment operator assigns the value of the right operand to the left operand.

```cpp
int a = 10;
```

## Compound Assignment Operators

Compound assignment operators perform an operation and then assign the result to the left operand.

| Operator | Equivalent to |
|----------|---------------|
| `+=`     | `a = a + b`   |
| `-=`     | `a = a - b`   |
| `*=`     | `a = a * b`   |
| `/=`     | `a = a / b`   |
| `%=`     | `a = a % b`   |
| `&=`     | `a = a & b`   |
| `|=`     | `a = a | b`   |
| `^=`     | `a = a ^ b`   |
| `<<=`    | `a = a << b`  |
| `>>=`    | `a = a >> b`  |

### Example

```cpp
#include <iostream>

int main() {
    int a = 10;
    a += 5; // a is now 15
    std::cout << "a += 5: " << a << std::endl;

    int b = 20;
    b -= 10; // b is now 10
    std::cout << "b -= 10: " << b << std::endl;

    int c = 5;
    c *= 2; // c is now 10
    std::cout << "c *= 2: " << c << std::endl;

    return 0;
}
```
