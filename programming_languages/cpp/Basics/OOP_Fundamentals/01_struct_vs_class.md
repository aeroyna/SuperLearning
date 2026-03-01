# Struct vs Class in C++

One of the most common C++ interview questions. The answer is simpler than you might think!

## The Only Technical Difference

In C++, **struct and class are almost identical**. The only differences are:

| Aspect | struct | class |
|--------|--------|-------|
| Default member access | public | private |
| Default inheritance | public | private |

That's it! Everything else is the same.

## Examples

### Default Access

```cpp
struct MyStruct {
    int x;          // public by default
    void foo();     // public by default
};

class MyClass {
    int x;          // private by default
    void foo();     // private by default
};

// Equivalent to:
struct MyStruct {
public:
    int x;
    void foo();
};

class MyClass {
private:
    int x;
    void foo();
};
```

### Default Inheritance

```cpp
struct DerivedStruct : BaseClass {  // public inheritance by default
};

class DerivedClass : BaseClass {    // private inheritance by default
};

// Equivalent to:
struct DerivedStruct : public BaseClass { };
class DerivedClass : private BaseClass { };
```

## Everything Else Is the Same

Both can have:
- Constructors and destructors
- Member functions
- Static members
- Virtual functions
- Templates
- Access specifiers

```cpp
struct ComplexStruct {
private:
    int secret;

public:
    ComplexStruct(int s) : secret(s) {}
    virtual void foo() { }
    static int count;

    template<typename T>
    void bar(T t) { }
};

// This struct is functionally identical to a class
```

## When to Use Which (Convention)

While technically interchangeable, there are **conventions**:

### Use `struct` for:

1. **Plain Old Data (POD)** - simple data containers
```cpp
struct Point {
    double x, y;
};

struct Color {
    uint8_t r, g, b, a;
};
```

2. **Data Transfer Objects (DTOs)**
```cpp
struct UserDTO {
    std::string name;
    int age;
    std::string email;
};
```

3. **Functors** (function objects)
```cpp
struct Compare {
    bool operator()(int a, int b) const {
        return a < b;
    }
};
```

4. **When all members are public**

### Use `class` for:

1. **Complex objects with invariants**
```cpp
class BankAccount {
private:
    double balance;  // Must maintain: balance >= 0

public:
    void withdraw(double amount);  // Enforces invariant
};
```

2. **When encapsulation matters**
3. **Objects with significant behavior**
4. **When using inheritance hierarchies**

## C vs C++ struct

| Feature | C struct | C++ struct |
|---------|----------|------------|
| Member functions | No | Yes |
| Access specifiers | No | Yes |
| Constructors | No | Yes |
| Inheritance | No | Yes |
| Templates | No | Yes |

```cpp
// C - only data
struct CStyle {
    int x;
    int y;
};

// C++ - full featured
struct CppStyle {
    int x, y;

    CppStyle(int a, int b) : x(a), y(b) {}
    int sum() const { return x + y; }
};
```

## Interview Deep Dive

### "Why does C++ have both?"

Historical reasons:
1. C compatibility - `struct` keyword kept for C code
2. `class` introduced to signal OOP intent
3. Default access difference emphasizes typical use

### "Which should I use for a template parameter?"

Convention: Use `class` or `typename` for template parameters:
```cpp
template<class T>     // Common
template<typename T>  // Also common
template<struct T>    // NOT valid! Can't use struct here
```

## Code Style Examples

### Google Style Guide approach:
```cpp
// struct for passive data
struct TableEntry {
    std::string key;
    int value;
};

// class for active objects
class Table {
public:
    void insert(const TableEntry& entry);
private:
    std::vector<TableEntry> entries_;
};
```

## Key Takeaways

- Technically: struct defaults to public, class to private
- Conventionally: struct for data, class for objects with behavior
- Both support all C++ features
- Use conventions consistently in your codebase

## Common Interview Questions

> [!question]- What's the difference between struct and class in C++?
> The only difference is default access: struct is public by default, class is private. Everything else (constructors, virtual functions, templates, etc.) is identical.

> [!question]- When would you choose struct over class?
> Use struct for plain data structures (POD), functors, and when all members are public. Use class when encapsulation, invariants, and complex behavior are involved.

> [!question]- Can a struct have private members?
> Yes! You can use `private:` in a struct just like in a class. It just defaults to public.
