# Stream Operator Overloading (`<<` and `>>`)

Stream operators allow your custom classes to work seamlessly with `std::cout`, `std::cin`, and file streams. This is one of the most commonly overloaded operators.

## Output Stream Operator (`<<`)

The insertion operator `<<` is used for output (e.g., `std::cout << obj`).

### Why It Must Be a Non-Member Function

```cpp
std::cout << myObject;
// This is equivalent to:
operator<<(std::cout, myObject);
// NOT:
std::cout.operator<<(myObject);  // We can't modify std::ostream!
```

Since we can't add member functions to `std::ostream`, we must use a non-member function.

### Basic Implementation

```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name;
    int age;

public:
    Person(const std::string& n, int a) : name(n), age(a) {}

    // Declare as friend to access private members
    friend std::ostream& operator<<(std::ostream& os, const Person& p);
};

// Definition (usually in .cpp file)
std::ostream& operator<<(std::ostream& os, const Person& p) {
    os << "Person(" << p.name << ", " << p.age << ")";
    return os;  // Return stream for chaining
}

int main() {
    Person alice("Alice", 30);
    std::cout << alice << std::endl;  // Person(Alice, 30)

    // Chaining works because we return the stream
    Person bob("Bob", 25);
    std::cout << alice << " and " << bob << std::endl;

    return 0;
}
```

### Why Return `std::ostream&`?

Returning the stream reference enables chaining:

```cpp
std::cout << a << b << c;
// Is evaluated as:
((std::cout << a) << b) << c;
// Each << returns std::cout, allowing the next <<
```

## Input Stream Operator (`>>`)

The extraction operator `>>` is used for input (e.g., `std::cin >> obj`).

```cpp
#include <iostream>
#include <string>

class Point {
private:
    double x, y;

public:
    Point(double x = 0, double y = 0) : x(x), y(y) {}

    friend std::ostream& operator<<(std::ostream& os, const Point& p);
    friend std::istream& operator>>(std::istream& is, Point& p);
};

std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << "(" << p.x << ", " << p.y << ")";
    return os;
}

std::istream& operator>>(std::istream& is, Point& p) {
    // Read format: x y
    is >> p.x >> p.y;

    // Optional: validate input
    if (!is) {
        p = Point();  // Reset to default on error
    }
    return is;
}

int main() {
    Point p;
    std::cout << "Enter x and y coordinates: ";
    std::cin >> p;
    std::cout << "You entered: " << p << std::endl;

    return 0;
}
```

> [!warning] Note on `operator>>`
> The `Point& p` parameter is NOT `const` because we're modifying the object with input data.

## Handling Complex Input Formats

```cpp
class Date {
private:
    int day, month, year;

public:
    Date(int d = 1, int m = 1, int y = 2000) : day(d), month(m), year(y) {}

    friend std::ostream& operator<<(std::ostream& os, const Date& d);
    friend std::istream& operator>>(std::istream& is, Date& d);
};

std::ostream& operator<<(std::ostream& os, const Date& d) {
    os << d.day << "/" << d.month << "/" << d.year;
    return os;
}

std::istream& operator>>(std::istream& is, Date& d) {
    char sep1, sep2;
    // Read format: DD/MM/YYYY
    is >> d.day >> sep1 >> d.month >> sep2 >> d.year;

    // Validate format
    if (sep1 != '/' || sep2 != '/') {
        is.setstate(std::ios::failbit);
    }
    return is;
}

int main() {
    Date d;
    std::cout << "Enter date (DD/MM/YYYY): ";
    std::cin >> d;

    if (std::cin) {
        std::cout << "Date: " << d << std::endl;
    } else {
        std::cout << "Invalid format!" << std::endl;
    }
}
```

## Working with File Streams

The same operators work with file streams:

```cpp
#include <fstream>

class Record {
    int id;
    std::string data;

public:
    Record(int i = 0, const std::string& d = "") : id(i), data(d) {}

    friend std::ostream& operator<<(std::ostream& os, const Record& r) {
        os << r.id << " " << r.data;
        return os;
    }

    friend std::istream& operator>>(std::istream& is, Record& r) {
        is >> r.id >> r.data;
        return is;
    }
};

int main() {
    // Write to file
    std::ofstream outFile("data.txt");
    Record r1(1, "Hello");
    outFile << r1 << std::endl;
    outFile.close();

    // Read from file
    std::ifstream inFile("data.txt");
    Record r2;
    inFile >> r2;
    std::cout << r2 << std::endl;

    return 0;
}
```

## Alternative: Public Getter or Print Method

If you don't want to use `friend`:

```cpp
class Person {
private:
    std::string name;
    int age;

public:
    // Option 1: Public print method
    std::ostream& print(std::ostream& os) const {
        os << "Person(" << name << ", " << age << ")";
        return os;
    }

    // Option 2: Public getters
    const std::string& getName() const { return name; }
    int getAge() const { return age; }
};

// Using print method
std::ostream& operator<<(std::ostream& os, const Person& p) {
    return p.print(os);
}

// Using getters
std::ostream& operator<<(std::ostream& os, const Person& p) {
    os << "Person(" << p.getName() << ", " << p.getAge() << ")";
    return os;
}
```

## Key Takeaways

- Stream operators must be non-member functions
- Usually declared as `friend` to access private members
- Always return the stream reference for chaining
- `operator<<` takes `const` object (reading)
- `operator>>` takes non-const object (writing)
- Check stream state after input for error handling

## Common Interview Questions

> [!question]- Why must `operator<<` be a non-member function?
> Because the left operand is `std::ostream`, not our class. We can't add member functions to `std::ostream`.

> [!question]- Why return `std::ostream&` instead of `void`?
> To enable chaining like `std::cout << a << b << c`. Each `<<` returns the stream for the next operation.

## Related Topics

- [[../../Intermediate/File_IO/01_file_streams|File Streams]]
- [[00_operator_overloading|Operator Overloading Overview]]
