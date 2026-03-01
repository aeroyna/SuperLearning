# static_cast

`static_cast` is the most commonly used cast in C++. It performs compile-time type conversions for related types.

## Syntax

```cpp
static_cast<target_type>(expression)
```

## Valid Uses

### 1. Numeric Type Conversions

```cpp
int i = 42;
double d = static_cast<double>(i);  // int to double

double pi = 3.14159;
int truncated = static_cast<int>(pi);  // 3 (truncates)

// Without cast, you might get warnings
double x = 10.5;
int y = x;  // Warning: possible loss of data
int z = static_cast<int>(x);  // OK: explicit intent
```

### 2. Pointer Conversions in Class Hierarchy

```cpp
class Base {};
class Derived : public Base {};

// Upcast (always safe, implicit)
Derived* d = new Derived();
Base* b = d;  // Implicit upcast

// Downcast (only safe if you KNOW the actual type)
Base* b2 = new Derived();
Derived* d2 = static_cast<Derived*>(b2);  // Works, but no runtime check!

// DANGER: If b2 actually pointed to Base, this would be UB
Base* b3 = new Base();
Derived* d3 = static_cast<Derived*>(b3);  // Compiles, but UB!
```

> [!warning] Warning
> `static_cast` does NOT perform runtime type checking. For safe downcasting with polymorphic types, use `dynamic_cast`.

### 3. Void Pointer Conversions

```cpp
int x = 42;
void* vp = &x;  // Any pointer to void* is implicit

int* ip = static_cast<int*>(vp);  // void* to specific type
std::cout << *ip << std::endl;  // 42
```

### 4. Enum to Integer

```cpp
enum class Color { Red, Green, Blue };

Color c = Color::Green;
int value = static_cast<int>(c);  // 1

// Integer to enum
int num = 2;
Color c2 = static_cast<Color>(num);  // Color::Blue
```

### 5. User-Defined Conversions

```cpp
class MyInt {
    int value;
public:
    MyInt(int v) : value(v) {}

    // Conversion operator
    explicit operator int() const { return value; }
};

MyInt mi(42);
int i = static_cast<int>(mi);  // Calls operator int()
```

### 6. Calling Explicit Constructors

```cpp
class Explicit {
public:
    explicit Explicit(int x) {}
};

// Explicit e = 42;  // Error: implicit conversion
Explicit e = static_cast<Explicit>(42);  // OK
```

## What static_cast CANNOT Do

```cpp
// Cannot cast away const
const int* cp = new int(42);
// int* p = static_cast<int*>(cp);  // Error! Use const_cast

// Cannot convert between unrelated types
int* ip = new int(42);
// double* dp = static_cast<double*>(ip);  // Error! Use reinterpret_cast

// Cannot downcast without inheritance relationship
class A {};
class B {};
A* a = new A();
// B* b = static_cast<B*>(a);  // Error: A and B are unrelated
```

## static_cast vs Implicit Conversion

```cpp
// Both work for widening conversions
int i = 10;
double d1 = i;                    // Implicit
double d2 = static_cast<double>(i);  // Explicit

// static_cast suppresses warnings for narrowing
double d = 3.14;
int i1 = d;                    // Warning: possible loss of data
int i2 = static_cast<int>(d);  // OK: explicit intent shown
```

## static_cast with References

```cpp
class Base {};
class Derived : public Base {};

Derived d;
Base& b = d;

// Downcast reference
Derived& d2 = static_cast<Derived&>(b);

// Failed cast throws... wait, no it doesn't!
// static_cast with references doesn't check, UB if wrong type
```

## Performance

`static_cast` has **zero runtime cost** for most conversions. The cast happens at compile time.

```cpp
// These compile to the same machine code:
int i = 42;
double d1 = i;
double d2 = static_cast<double>(i);
```

## Key Takeaways

- Most common cast for type conversions
- Performs compile-time checking
- Use for numeric conversions, void*, enum↔int
- For inheritance: safe for upcasting, UNSAFE for downcasting
- Zero runtime overhead
- Cannot cast away const or between unrelated pointer types

## Common Interview Questions

> [!question]- When is static_cast unsafe?
> When downcasting in an inheritance hierarchy. If the object isn't actually of the derived type, you get undefined behavior. Use `dynamic_cast` for safe downcasting.

> [!question]- What's the difference between `static_cast<int>(3.14)` and `(int)3.14`?
> For this simple case, they behave the same. But `static_cast` is preferred because it only performs valid conversions, while C-style casts might silently perform dangerous conversions.

## Related Topics

- [[02_dynamic_cast|dynamic_cast (for safe downcasting)]]
- [[05_c_style_vs_cpp_casts|C-Style vs C++ Casts]]
