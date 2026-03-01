# Practice Problems: Standard Template Library (STL)

## Problem 1: Word Frequency Counter

Write a program that reads a text from a string, and counts the frequency of each word. Use a `std::map` to store the word frequencies.

### Solution

```cpp
#include <iostream>
#include <string>
#include <map>
#include <sstream>

int main() {
    std::string text = "this is a test this is only a test";
    std::map<std::string, int> word_counts;
    std::stringstream ss(text);
    std::string word;

    while (ss >> word) {
        word_counts[word]++;
    }

    for (const auto& pair : word_counts) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}
```

## Problem 2: Remove duplicates from a vector

Write a function that takes a `std::vector<int>` and removes the duplicate elements.

### Solution using `std::set`

```cpp
#include <iostream>
#include <vector>
#include <set>

void remove_duplicates(std::vector<int>& vec) {
    std::set<int> s(vec.begin(), vec.end());
    vec.assign(s.begin(), s.end());
}

int main() {
    std::vector<int> numbers = {1, 2, 2, 3, 1, 4, 5, 4};
    remove_duplicates(numbers);

    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### Solution using `std::sort` and `std::unique`

This is a more efficient solution that modifies the vector in-place.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

void remove_duplicates_inplace(std::vector<int>& vec) {
    std::sort(vec.begin(), vec.end());
    auto last = std::unique(vec.begin(), vec.end());
    vec.erase(last, vec.end());
}

int main() {
    std::vector<int> numbers = {1, 2, 2, 3, 1, 4, 5, 4};
    remove_duplicates_inplace(numbers);

    for (int x : numbers) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

## Problem 3: Check for Palindrome

Write a function that checks if a `std::string` is a palindrome using STL algorithms. A palindrome is a word that reads the same forwards and backwards (e.g., "racecar").

### Solution

```cpp
#include <iostream>
#include <string>
#include <algorithm>

bool is_palindrome(const std::string& s) {
    std::string reversed_s = s;
    std::reverse(reversed_s.begin(), reversed_s.end());
    return s == reversed_s;
}

// More efficient solution without creating a new string
bool is_palindrome_efficient(const std::string& s) {
    return std::equal(s.begin(), s.begin() + s.size()/2, s.rbegin());
}

int main() {
    std::string s1 = "racecar";
    std::string s2 = "hello";

    std::cout << s1 << " is a palindrome: " << std::boolalpha << is_palindrome_efficient(s1) << std::endl;
    std::cout << s2 << " is a palindrome: " << std::boolalpha << is_palindrome_efficient(s2) << std::endl;

    return 0;
}
```
