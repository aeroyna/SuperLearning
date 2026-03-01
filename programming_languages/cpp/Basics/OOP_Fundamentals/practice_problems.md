# Practice Problems: OOP Fundamentals

## Problem 1: Const Correctness ⭐ (Easy)

Fix the const-correctness issues in this class:

```cpp
class Student {
    std::string name;
    int age;

public:
    Student(std::string n, int a) : name(n), age(a) {}

    std::string getName() { return name; }
    int getAge() { return age; }
    void setAge(int a) { age = a; }

    void print() {
        std::cout << name << ", " << age << std::endl;
    }
};

void displayStudent(Student& s) {
    std::cout << s.getName() << std::endl;
}
```

> [!success]- Solution
> ```cpp
> class Student {
>     std::string name;
>     int age;
>
> public:
>     Student(const std::string& n, int a) : name(n), age(a) {}
>
>     const std::string& getName() const { return name; }  // const method, return const ref
>     int getAge() const { return age; }  // const method
>     void setAge(int a) { age = a; }  // OK - non-const (modifies)
>
>     void print() const {  // const method
>         std::cout << name << ", " << age << std::endl;
>     }
> };
>
> void displayStudent(const Student& s) {  // const reference
>     std::cout << s.getName() << std::endl;
> }
> ```

---

## Problem 2: Static Member Counter ⭐⭐ (Medium)

Implement a `Connection` class that:
- Tracks total number of active connections (static)
- Has a maximum connection limit (static const)
- Prevents creating connections beyond the limit
- Properly decrements count when connections are destroyed

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <stdexcept>
>
> class Connection {
> private:
>     static int activeCount;
>     static const int MAX_CONNECTIONS = 10;
>     int id;
>
> public:
>     Connection() {
>         if (activeCount >= MAX_CONNECTIONS) {
>             throw std::runtime_error("Connection limit exceeded");
>         }
>         id = ++activeCount;
>         std::cout << "Connection " << id << " opened\n";
>     }
>
>     ~Connection() {
>         std::cout << "Connection " << id << " closed\n";
>         --activeCount;
>     }
>
>     // Delete copy to prevent count issues
>     Connection(const Connection&) = delete;
>     Connection& operator=(const Connection&) = delete;
>
>     // Allow move
>     Connection(Connection&& other) noexcept : id(other.id) {
>         other.id = 0;  // Mark as moved
>     }
>
>     static int getActiveCount() { return activeCount; }
>     static int getMaxConnections() { return MAX_CONNECTIONS; }
> };
>
> int Connection::activeCount = 0;
>
> int main() {
>     std::cout << "Active: " << Connection::getActiveCount() << "\n";
>
>     {
>         Connection c1;
>         Connection c2;
>         std::cout << "Active: " << Connection::getActiveCount() << "\n";
>     }
>
>     std::cout << "Active: " << Connection::getActiveCount() << "\n";
>     return 0;
> }
> ```

---

## Problem 3: Friend for Comparison ⭐⭐ (Medium)

Implement a `Money` class with:
- Private amount in cents
- `operator==` and `operator<` as friend functions
- `operator<<` for output

> [!success]- Solution
> ```cpp
> #include <iostream>
>
> class Money {
> private:
>     long cents;
>
> public:
>     Money(long dollars = 0, long c = 0) : cents(dollars * 100 + c) {}
>
>     friend bool operator==(const Money& a, const Money& b) {
>         return a.cents == b.cents;
>     }
>
>     friend bool operator!=(const Money& a, const Money& b) {
>         return !(a == b);
>     }
>
>     friend bool operator<(const Money& a, const Money& b) {
>         return a.cents < b.cents;
>     }
>
>     friend bool operator>(const Money& a, const Money& b) {
>         return b < a;
>     }
>
>     friend bool operator<=(const Money& a, const Money& b) {
>         return !(b < a);
>     }
>
>     friend bool operator>=(const Money& a, const Money& b) {
>         return !(a < b);
>     }
>
>     friend std::ostream& operator<<(std::ostream& os, const Money& m) {
>         os << "$" << (m.cents / 100) << "."
>            << (m.cents % 100 < 10 ? "0" : "") << (m.cents % 100);
>         return os;
>     }
> };
>
> int main() {
>     Money m1(10, 50);  // $10.50
>     Money m2(10, 50);
>     Money m3(20, 0);
>
>     std::cout << m1 << " == " << m2 << ": " << (m1 == m2) << "\n";
>     std::cout << m1 << " < " << m3 << ": " << (m1 < m3) << "\n";
>
>     return 0;
> }
> ```

---

## Problem 4: Pointer Const Quiz ⭐ (Easy)

Which lines compile? What do they mean?

```cpp
int x = 10, y = 20;

const int* a = &x;      // Line 1
int const* b = &x;      // Line 2
int* const c = &x;      // Line 3
const int* const d = &x; // Line 4

*a = 30;                // Line 5
a = &y;                 // Line 6
*c = 30;                // Line 7
c = &y;                 // Line 8
```

> [!success]- Solution
> ```
> Line 1: Compiles. a is pointer to const int (can change a, not *a)
> Line 2: Compiles. Same as Line 1 (const int* == int const*)
> Line 3: Compiles. c is const pointer to int (can change *c, not c)
> Line 4: Compiles. d is const pointer to const int (can't change either)
> Line 5: ERROR - *a is const
> Line 6: Compiles - a itself can change
> Line 7: Compiles - *c can change
> Line 8: ERROR - c itself is const
> ```

---

## Problem 5: Struct vs Class ⭐ (Easy)

What's the output?

```cpp
struct S {
    int x;
    void print() { std::cout << x << " "; }
};

class C {
    int x;
    void print() { std::cout << x << " "; }
};

int main() {
    S s;
    s.x = 10;
    s.print();

    C c;
    c.x = 20;
    c.print();

    return 0;
}
```

> [!success]- Solution
> The code does not compile!
>
> - `S` struct: `x` and `print()` are public by default - `s.x = 10` and `s.print()` work
> - `C` class: `x` and `print()` are private by default - `c.x = 20` and `c.print()` cause errors
>
> Fixed version for C:
> ```cpp
> class C {
> public:  // Need this!
>     int x;
>     void print() { std::cout << x << " "; }
> };
> ```

---

## Problem 6: Mutable for Caching ⭐⭐ (Medium)

Implement a `Circle` class with:
- Radius as the only non-mutable member
- Area calculation that caches the result
- The cached area should be recalculated only when radius changes

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <cmath>
>
> class Circle {
> private:
>     double radius;
>     mutable double cachedArea;
>     mutable bool areaCacheValid;
>
> public:
>     Circle(double r) : radius(r), cachedArea(0), areaCacheValid(false) {}
>
>     double getRadius() const { return radius; }
>
>     void setRadius(double r) {
>         if (r != radius) {
>             radius = r;
>             areaCacheValid = false;  // Invalidate cache
>         }
>     }
>
>     double getArea() const {  // const method, but can update cache
>         if (!areaCacheValid) {
>             std::cout << "(calculating area) ";
>             cachedArea = M_PI * radius * radius;
>             areaCacheValid = true;
>         }
>         return cachedArea;
>     }
> };
>
> int main() {
>     Circle c(5.0);
>
>     std::cout << "Area: " << c.getArea() << "\n";  // Calculates
>     std::cout << "Area: " << c.getArea() << "\n";  // Uses cache
>
>     c.setRadius(10.0);
>     std::cout << "Area: " << c.getArea() << "\n";  // Recalculates
>     std::cout << "Area: " << c.getArea() << "\n";  // Uses cache
>
>     return 0;
> }
> ```

---

## Problem 7: Singleton with Static ⭐⭐⭐ (Hard)

Implement a thread-safe Logger singleton using:
- Private constructor
- Static `getInstance()` method
- Deleted copy/move operations

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <string>
> #include <mutex>
>
> class Logger {
> private:
>     Logger() { std::cout << "Logger created\n"; }
>
>     // Private destructor - prevents deletion except by Logger itself
>     ~Logger() { std::cout << "Logger destroyed\n"; }
>
> public:
>     // Delete copy and move
>     Logger(const Logger&) = delete;
>     Logger& operator=(const Logger&) = delete;
>     Logger(Logger&&) = delete;
>     Logger& operator=(Logger&&) = delete;
>
>     // Thread-safe singleton access (Meyer's Singleton)
>     static Logger& getInstance() {
>         static Logger instance;  // C++11: thread-safe initialization
>         return instance;
>     }
>
>     void log(const std::string& message) {
>         std::lock_guard<std::mutex> lock(mutex_);
>         std::cout << "[LOG] " << message << "\n";
>     }
>
> private:
>     std::mutex mutex_;
> };
>
> int main() {
>     Logger::getInstance().log("Application started");
>     Logger::getInstance().log("Processing...");
>
>     // Logger& l1 = Logger::getInstance();
>     // Logger l2 = l1;  // Error: copy deleted
>
>     return 0;
> }
> ```
