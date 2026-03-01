# Practice Problems: Operator Overloading

## Problem 1: Fraction Class ⭐⭐ (Medium)

Create a `Fraction` class that represents a fraction with numerator and denominator. Implement the following operators:
- Arithmetic: `+`, `-`, `*`, `/`
- Comparison: `==`, `!=`, `<`
- Stream: `<<`
- The fraction should always be stored in reduced form (use GCD)

> [!hint]- Hint
> Use the Euclidean algorithm for GCD. Implement compound operators first, then binary operators using them.

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <numeric>  // for std::gcd in C++17
>
> class Fraction {
> private:
>     int num, den;
>
>     void reduce() {
>         if (den < 0) { num = -num; den = -den; }
>         int g = std::gcd(std::abs(num), std::abs(den));
>         num /= g;
>         den /= g;
>     }
>
> public:
>     Fraction(int n = 0, int d = 1) : num(n), den(d) {
>         if (den == 0) throw std::invalid_argument("Denominator cannot be 0");
>         reduce();
>     }
>
>     Fraction& operator+=(const Fraction& other) {
>         num = num * other.den + other.num * den;
>         den = den * other.den;
>         reduce();
>         return *this;
>     }
>
>     Fraction& operator-=(const Fraction& other) {
>         num = num * other.den - other.num * den;
>         den = den * other.den;
>         reduce();
>         return *this;
>     }
>
>     Fraction& operator*=(const Fraction& other) {
>         num *= other.num;
>         den *= other.den;
>         reduce();
>         return *this;
>     }
>
>     Fraction& operator/=(const Fraction& other) {
>         num *= other.den;
>         den *= other.num;
>         reduce();
>         return *this;
>     }
>
>     friend Fraction operator+(Fraction lhs, const Fraction& rhs) {
>         return lhs += rhs;
>     }
>
>     friend Fraction operator-(Fraction lhs, const Fraction& rhs) {
>         return lhs -= rhs;
>     }
>
>     friend Fraction operator*(Fraction lhs, const Fraction& rhs) {
>         return lhs *= rhs;
>     }
>
>     friend Fraction operator/(Fraction lhs, const Fraction& rhs) {
>         return lhs /= rhs;
>     }
>
>     bool operator==(const Fraction& other) const {
>         return num == other.num && den == other.den;
>     }
>
>     bool operator!=(const Fraction& other) const {
>         return !(*this == other);
>     }
>
>     bool operator<(const Fraction& other) const {
>         return num * other.den < other.num * den;
>     }
>
>     friend std::ostream& operator<<(std::ostream& os, const Fraction& f) {
>         os << f.num;
>         if (f.den != 1) os << "/" << f.den;
>         return os;
>     }
> };
>
> int main() {
>     Fraction a(1, 2);
>     Fraction b(1, 3);
>
>     std::cout << a << " + " << b << " = " << (a + b) << std::endl;  // 5/6
>     std::cout << a << " * " << b << " = " << (a * b) << std::endl;  // 1/6
>     std::cout << (a < b) << std::endl;  // 0 (false)
>
>     Fraction c(2, 4);
>     std::cout << (a == c) << std::endl;  // 1 (true, 1/2 == 2/4 reduced)
>
>     return 0;
> }
> ```

---

## Problem 2: Vector2D Class ⭐⭐ (Medium)

Create a 2D vector class with:
- Arithmetic: `+`, `-`, `*` (scalar multiplication), unary `-`
- Comparison: `==`
- Subscript: `[]` for x (index 0) and y (index 1)
- Stream: `<<`
- Member function: `magnitude()` to get vector length

> [!hint]- Hint
> Remember to provide both const and non-const versions of `operator[]`.

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <cmath>
> #include <stdexcept>
>
> class Vector2D {
> private:
>     double x, y;
>
> public:
>     Vector2D(double x = 0, double y = 0) : x(x), y(y) {}
>
>     // Arithmetic operators
>     Vector2D operator+(const Vector2D& other) const {
>         return Vector2D(x + other.x, y + other.y);
>     }
>
>     Vector2D operator-(const Vector2D& other) const {
>         return Vector2D(x - other.x, y - other.y);
>     }
>
>     Vector2D operator-() const {
>         return Vector2D(-x, -y);
>     }
>
>     // Scalar multiplication
>     Vector2D operator*(double scalar) const {
>         return Vector2D(x * scalar, y * scalar);
>     }
>
>     // Allow scalar * vector
>     friend Vector2D operator*(double scalar, const Vector2D& v) {
>         return v * scalar;
>     }
>
>     // Comparison
>     bool operator==(const Vector2D& other) const {
>         return x == other.x && y == other.y;
>     }
>
>     // Subscript operators
>     double& operator[](size_t index) {
>         if (index == 0) return x;
>         if (index == 1) return y;
>         throw std::out_of_range("Index must be 0 or 1");
>     }
>
>     const double& operator[](size_t index) const {
>         if (index == 0) return x;
>         if (index == 1) return y;
>         throw std::out_of_range("Index must be 0 or 1");
>     }
>
>     // Stream operator
>     friend std::ostream& operator<<(std::ostream& os, const Vector2D& v) {
>         os << "(" << v.x << ", " << v.y << ")";
>         return os;
>     }
>
>     double magnitude() const {
>         return std::sqrt(x * x + y * y);
>     }
> };
>
> int main() {
>     Vector2D v1(3, 4);
>     Vector2D v2(1, 2);
>
>     std::cout << "v1 = " << v1 << std::endl;
>     std::cout << "v2 = " << v2 << std::endl;
>     std::cout << "v1 + v2 = " << (v1 + v2) << std::endl;
>     std::cout << "v1 * 2 = " << (v1 * 2) << std::endl;
>     std::cout << "2 * v1 = " << (2 * v1) << std::endl;
>     std::cout << "-v1 = " << (-v1) << std::endl;
>     std::cout << "v1[0] = " << v1[0] << std::endl;
>     std::cout << "|v1| = " << v1.magnitude() << std::endl;  // 5
>
>     v1[0] = 10;
>     std::cout << "After v1[0] = 10: " << v1 << std::endl;
>
>     return 0;
> }
> ```

---

## Problem 3: String Class with Full Operators ⭐⭐⭐ (Hard)

Implement a simplified string class with:
- Copy constructor, copy assignment (Rule of Three)
- Move constructor, move assignment (Rule of Five)
- `+` for concatenation
- `[]` for character access
- `==`, `<` for comparison
- `<<` for output

> [!hint]- Hint
> Use copy-and-swap idiom for exception-safe assignment. Mark move operations as `noexcept`.

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <cstring>
> #include <algorithm>
>
> class String {
> private:
>     char* data;
>     size_t len;
>
> public:
>     // Default constructor
>     String(const char* s = "") {
>         len = strlen(s);
>         data = new char[len + 1];
>         strcpy(data, s);
>     }
>
>     // Copy constructor
>     String(const String& other) {
>         len = other.len;
>         data = new char[len + 1];
>         strcpy(data, other.data);
>     }
>
>     // Move constructor
>     String(String&& other) noexcept : data(other.data), len(other.len) {
>         other.data = nullptr;
>         other.len = 0;
>     }
>
>     // Destructor
>     ~String() {
>         delete[] data;
>     }
>
>     // Swap function
>     void swap(String& other) noexcept {
>         std::swap(data, other.data);
>         std::swap(len, other.len);
>     }
>
>     // Copy-and-swap assignment (handles both copy and move)
>     String& operator=(String other) noexcept {
>         swap(other);
>         return *this;
>     }
>
>     // Concatenation
>     String operator+(const String& other) const {
>         String result;
>         delete[] result.data;
>         result.len = len + other.len;
>         result.data = new char[result.len + 1];
>         strcpy(result.data, data);
>         strcat(result.data, other.data);
>         return result;
>     }
>
>     String& operator+=(const String& other) {
>         *this = *this + other;
>         return *this;
>     }
>
>     // Subscript operators
>     char& operator[](size_t index) {
>         return data[index];
>     }
>
>     const char& operator[](size_t index) const {
>         return data[index];
>     }
>
>     // Comparison operators
>     bool operator==(const String& other) const {
>         return strcmp(data, other.data) == 0;
>     }
>
>     bool operator!=(const String& other) const {
>         return !(*this == other);
>     }
>
>     bool operator<(const String& other) const {
>         return strcmp(data, other.data) < 0;
>     }
>
>     // Stream operator
>     friend std::ostream& operator<<(std::ostream& os, const String& s) {
>         os << s.data;
>         return os;
>     }
>
>     size_t length() const { return len; }
>     const char* c_str() const { return data; }
> };
>
> int main() {
>     String s1("Hello");
>     String s2(" World");
>     String s3 = s1 + s2;
>
>     std::cout << "s1: " << s1 << std::endl;
>     std::cout << "s2: " << s2 << std::endl;
>     std::cout << "s3: " << s3 << std::endl;
>
>     std::cout << "s3[0]: " << s3[0] << std::endl;
>
>     String s4 = s1;  // Copy
>     std::cout << "s4 == s1: " << (s4 == s1) << std::endl;
>
>     String s5 = std::move(s4);  // Move
>     std::cout << "s5: " << s5 << std::endl;
>
>     return 0;
> }
> ```

---

## Problem 4: Matrix Class ⭐⭐⭐ (Hard)

Create a matrix class with:
- `()` operator for element access: `m(row, col)`
- `+` and `-` for matrix addition/subtraction
- `*` for matrix multiplication
- `<<` for output

> [!hint]- Hint
> Store data in a 1D array with row-major ordering. For multiplication, the inner dimensions must match.

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <vector>
> #include <stdexcept>
> #include <iomanip>
>
> class Matrix {
> private:
>     std::vector<double> data;
>     size_t rows, cols;
>
> public:
>     Matrix(size_t r, size_t c, double init = 0)
>         : rows(r), cols(c), data(r * c, init) {}
>
>     // Element access
>     double& operator()(size_t r, size_t c) {
>         return data[r * cols + c];
>     }
>
>     const double& operator()(size_t r, size_t c) const {
>         return data[r * cols + c];
>     }
>
>     // Addition
>     Matrix operator+(const Matrix& other) const {
>         if (rows != other.rows || cols != other.cols)
>             throw std::invalid_argument("Matrix dimensions must match");
>
>         Matrix result(rows, cols);
>         for (size_t i = 0; i < data.size(); ++i)
>             result.data[i] = data[i] + other.data[i];
>         return result;
>     }
>
>     // Subtraction
>     Matrix operator-(const Matrix& other) const {
>         if (rows != other.rows || cols != other.cols)
>             throw std::invalid_argument("Matrix dimensions must match");
>
>         Matrix result(rows, cols);
>         for (size_t i = 0; i < data.size(); ++i)
>             result.data[i] = data[i] - other.data[i];
>         return result;
>     }
>
>     // Multiplication
>     Matrix operator*(const Matrix& other) const {
>         if (cols != other.rows)
>             throw std::invalid_argument("Invalid dimensions for multiplication");
>
>         Matrix result(rows, other.cols);
>         for (size_t i = 0; i < rows; ++i) {
>             for (size_t j = 0; j < other.cols; ++j) {
>                 double sum = 0;
>                 for (size_t k = 0; k < cols; ++k) {
>                     sum += (*this)(i, k) * other(k, j);
>                 }
>                 result(i, j) = sum;
>             }
>         }
>         return result;
>     }
>
>     // Stream output
>     friend std::ostream& operator<<(std::ostream& os, const Matrix& m) {
>         for (size_t i = 0; i < m.rows; ++i) {
>             os << "[ ";
>             for (size_t j = 0; j < m.cols; ++j) {
>                 os << std::setw(8) << m(i, j) << " ";
>             }
>             os << "]\n";
>         }
>         return os;
>     }
>
>     size_t getRows() const { return rows; }
>     size_t getCols() const { return cols; }
> };
>
> int main() {
>     Matrix a(2, 3);
>     a(0, 0) = 1; a(0, 1) = 2; a(0, 2) = 3;
>     a(1, 0) = 4; a(1, 1) = 5; a(1, 2) = 6;
>
>     Matrix b(3, 2);
>     b(0, 0) = 7; b(0, 1) = 8;
>     b(1, 0) = 9; b(1, 1) = 10;
>     b(2, 0) = 11; b(2, 1) = 12;
>
>     std::cout << "Matrix A (2x3):\n" << a << std::endl;
>     std::cout << "Matrix B (3x2):\n" << b << std::endl;
>     std::cout << "A * B (2x2):\n" << (a * b) << std::endl;
>
>     return 0;
> }
> ```

---

## Problem 5: Smart Iterator ⭐⭐ (Medium)

Create a `Range` class that can be used in range-based for loops. Implement the iterator with increment operators.

Example usage:
```cpp
for (int x : Range(1, 10, 2)) {  // start=1, end=10, step=2
    std::cout << x << " ";  // 1 3 5 7 9
}
```

> [!success]- Solution
> ```cpp
> #include <iostream>
>
> class Range {
> public:
>     class Iterator {
>     private:
>         int current;
>         int step;
>
>     public:
>         Iterator(int val, int s) : current(val), step(s) {}
>
>         int operator*() const { return current; }
>
>         Iterator& operator++() {
>             current += step;
>             return *this;
>         }
>
>         Iterator operator++(int) {
>             Iterator temp = *this;
>             ++(*this);
>             return temp;
>         }
>
>         bool operator!=(const Iterator& other) const {
>             if (step > 0) return current < other.current;
>             return current > other.current;
>         }
>     };
>
> private:
>     int start_, end_, step_;
>
> public:
>     Range(int end) : start_(0), end_(end), step_(1) {}
>     Range(int start, int end, int step = 1)
>         : start_(start), end_(end), step_(step) {}
>
>     Iterator begin() { return Iterator(start_, step_); }
>     Iterator end() { return Iterator(end_, step_); }
> };
>
> int main() {
>     std::cout << "Range(5): ";
>     for (int x : Range(5))
>         std::cout << x << " ";
>     std::cout << std::endl;
>
>     std::cout << "Range(1, 10): ";
>     for (int x : Range(1, 10))
>         std::cout << x << " ";
>     std::cout << std::endl;
>
>     std::cout << "Range(0, 10, 2): ";
>     for (int x : Range(0, 10, 2))
>         std::cout << x << " ";
>     std::cout << std::endl;
>
>     std::cout << "Range(10, 0, -1): ";
>     for (int x : Range(10, 0, -1))
>         std::cout << x << " ";
>     std::cout << std::endl;
>
>     return 0;
> }
> ```
