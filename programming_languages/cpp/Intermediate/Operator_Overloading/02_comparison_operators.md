# Comparison Operator Overloading

Comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) allow objects to be compared and used in sorting algorithms, containers like `std::set`, and conditional statements.

## Basic Example

```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name;
    int age;

public:
    Person(const std::string& n, int a) : name(n), age(a) {}

    // Equality operator
    bool operator==(const Person& other) const {
        return name == other.name && age == other.age;
    }

    // Inequality operator (implement in terms of ==)
    bool operator!=(const Person& other) const {
        return !(*this == other);
    }

    // Less than (useful for sorting)
    bool operator<(const Person& other) const {
        if (age != other.age) return age < other.age;
        return name < other.name;  // secondary sort by name
    }

    // Greater than (implement in terms of <)
    bool operator>(const Person& other) const {
        return other < *this;
    }

    // Less than or equal
    bool operator<=(const Person& other) const {
        return !(other < *this);
    }

    // Greater than or equal
    bool operator>=(const Person& other) const {
        return !(*this < other);
    }
};

int main() {
    Person alice("Alice", 30);
    Person bob("Bob", 25);
    Person alice2("Alice", 30);

    std::cout << std::boolalpha;
    std::cout << "alice == alice2: " << (alice == alice2) << std::endl;  // true
    std::cout << "alice == bob: " << (alice == bob) << std::endl;        // false
    std::cout << "bob < alice: " << (bob < alice) << std::endl;          // true (25 < 30)

    return 0;
}
```

## The Canonical Way: Implement Only Two

You only need to implement `==` and `<`, then define the rest in terms of them:

```cpp
class MyClass {
public:
    bool operator==(const MyClass& other) const { /* ... */ }
    bool operator<(const MyClass& other) const { /* ... */ }

    // Derived operators
    bool operator!=(const MyClass& other) const { return !(*this == other); }
    bool operator>(const MyClass& other) const { return other < *this; }
    bool operator<=(const MyClass& other) const { return !(other < *this); }
    bool operator>=(const MyClass& other) const { return !(*this < other); }
};
```

## C++20: Spaceship Operator (`<=>`)

C++20 introduced the three-way comparison operator, which can generate all comparison operators automatically:

```cpp
#include <compare>

class Person {
private:
    std::string name;
    int age;

public:
    Person(const std::string& n, int a) : name(n), age(a) {}

    // Spaceship operator - generates all 6 comparison operators!
    auto operator<=>(const Person& other) const = default;
};

int main() {
    Person a("Alice", 30);
    Person b("Bob", 25);

    // All these work automatically:
    bool eq = (a == b);
    bool lt = (a < b);
    bool gt = (a > b);
    // etc.
}
```

### Custom Spaceship Implementation

```cpp
#include <compare>

class Version {
private:
    int major, minor, patch;

public:
    Version(int ma, int mi, int p) : major(ma), minor(mi), patch(p) {}

    std::strong_ordering operator<=>(const Version& other) const {
        if (auto cmp = major <=> other.major; cmp != 0) return cmp;
        if (auto cmp = minor <=> other.minor; cmp != 0) return cmp;
        return patch <=> other.patch;
    }

    bool operator==(const Version& other) const = default;
};
```

## Non-Member Implementation

For comparing with different types:

```cpp
class Distance {
private:
    double meters;

public:
    explicit Distance(double m) : meters(m) {}

    // Compare Distance with raw double
    friend bool operator==(const Distance& d, double m) {
        return d.meters == m;
    }

    friend bool operator==(double m, const Distance& d) {
        return d.meters == m;
    }
};

int main() {
    Distance d(100.0);
    if (d == 100.0) { /* ... */ }  // works
    if (100.0 == d) { /* ... */ }  // also works
}
```

## Using with STL Containers

`operator<` enables use with ordered containers:

```cpp
#include <set>
#include <map>
#include <algorithm>

class Student {
public:
    std::string name;
    int id;

    bool operator<(const Student& other) const {
        return id < other.id;
    }
};

int main() {
    std::set<Student> students;  // requires operator<
    students.insert({"Alice", 101});
    students.insert({"Bob", 102});

    std::map<Student, double> grades;  // key requires operator<
    grades[{"Alice", 101}] = 95.5;

    std::vector<Student> vec = {{"Bob", 102}, {"Alice", 101}};
    std::sort(vec.begin(), vec.end());  // uses operator<
}
```

## Key Takeaways

- Implement `==` and `<` first, derive others from them
- Always mark comparison operators as `const`
- Use `<=>` in C++20 to auto-generate all comparisons
- `operator<` is required for ordered STL containers (`set`, `map`)
- `operator==` is required for unordered containers (`unordered_set`, `unordered_map`) along with a hash function

## Common Interview Questions

> [!question]- What's the difference between `==` and `<=>` in C++20?
> `==` returns `bool`. `<=>` returns a comparison category (`strong_ordering`, `weak_ordering`, `partial_ordering`) that encodes less-than, equal, or greater-than in a single operation.

> [!question]- Why do `std::set` and `std::map` only need `operator<`?
> They use a strict weak ordering. Equality is determined by `!(a < b) && !(b < a)`. This is more efficient than requiring both `<` and `==`.

## Related Topics

- [[../Standard_Template_Library/Containers/04_set|std::set]]
- [[../Standard_Template_Library/Containers/03_map|std::map]]
