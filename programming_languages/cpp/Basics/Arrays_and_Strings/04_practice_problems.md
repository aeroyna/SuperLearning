# Practice Problems: Arrays and Strings

## Problem 1: Find the largest element in an array

Write a C++ program to find the largest element in an array of integers.

### Solution

```cpp
#include <iostream>

int main() {
    int numbers[] = {10, 5, 20, 8, 15};
    int n = sizeof(numbers) / sizeof(numbers[0]);
    int max = numbers[0];

    for (int i = 1; i < n; ++i) {
        if (numbers[i] > max) {
            max = numbers[i];
        }
    }

    std::cout << "The largest element is: " << max << std::endl;

    return 0;
}
```

## Problem 2: Reverse a string

Write a C++ program that reverses a `std::string`.

### Solution

```cpp
#include <iostream>
#include <string>
#include <algorithm>

int main() {
    std::string s = "hello";
    std::cout << "Original string: " << s << std::endl;

    std::reverse(s.begin(), s.end());

    std::cout << "Reversed string: " << s << std::endl;

    return 0;
}
```

### Manual Solution (without `<algorithm>`)

```cpp
#include <iostream>
#include <string>

int main() {
    std::string s = "hello";
    std::cout << "Original string: " << s << std::endl;

    int n = s.length();
    for (int i = 0; i < n / 2; ++i) {
        std::swap(s[i], s[n - i - 1]);
    }

    std::cout << "Reversed string: " << s << std::endl;

    return 0;
}
```

## Problem 3: Count vowels in a string

Write a C++ program that counts the number of vowels (a, e, i, o, u) in a `std::string`.

### Solution

```cpp
#include <iostream>
#include <string>

int main() {
    std::string s = "Hello, World!";
    int count = 0;

    for (char c : s) {
        c = tolower(c);
        if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u') {
            count++;
        }
    }

    std::cout << "The number of vowels is: " << count << std::endl;

    return 0;
}
```
