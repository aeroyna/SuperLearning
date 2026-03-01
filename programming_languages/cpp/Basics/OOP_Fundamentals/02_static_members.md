# Static Members

Static members belong to the **class itself**, not to individual objects. Understanding them is crucial for interviews and proper OOP design.

## Static vs Non-Static

| Aspect | Non-Static | Static |
|--------|-----------|--------|
| Belongs to | Each object | The class |
| Memory | Per instance | Single copy |
| Access | Need object | No object needed |
| `this` pointer | Available | NOT available |

## Static Member Variables

### Declaration and Definition

```cpp
class Counter {
public:
    static int count;  // Declaration (in class)
};

int Counter::count = 0;  // Definition (outside class, usually in .cpp)
```

### Usage

```cpp
class Counter {
public:
    static int count;

    Counter() { ++count; }
    ~Counter() { --count; }
};

int Counter::count = 0;

int main() {
    std::cout << Counter::count << std::endl;  // 0

    Counter c1;
    std::cout << Counter::count << std::endl;  // 1

    {
        Counter c2, c3;
        std::cout << Counter::count << std::endl;  // 3
    }

    std::cout << Counter::count << std::endl;  // 1 (c2, c3 destroyed)
}
```

### Inline Static (C++17)

```cpp
class Modern {
public:
    inline static int value = 42;  // C++17: define in class
    static inline std::string name = "Test";  // Both orders work
};
// No separate definition needed!
```

### Constexpr Static

```cpp
class Config {
public:
    static constexpr int MAX_SIZE = 100;     // OK: integral type
    static constexpr double PI = 3.14159;    // C++11: constexpr
    static inline const std::string NAME = "Config";  // C++17 for non-literal
};
```

## Static Member Functions

Static functions can only access static members (no `this` pointer).

```cpp
class Math {
private:
    static const double PI;

public:
    // Static function - no 'this', no object needed
    static double circleArea(double r) {
        return PI * r * r;
    }

    // Non-static function - needs object
    double instanceMethod() {
        return 0;  // Can access 'this'
    }
};

const double Math::PI = 3.14159;

int main() {
    // Call static function without object
    double area = Math::circleArea(5);

    // Can also call through object (unusual)
    Math m;
    double area2 = m.circleArea(5);  // Works, but misleading
}
```

### What Static Functions CANNOT Do

```cpp
class Example {
    int instanceVar;
    static int staticVar;

public:
    static void staticFunc() {
        // staticVar = 10;     // OK
        // instanceVar = 10;   // ERROR: no 'this' pointer
        // this->foo();        // ERROR: no 'this'
    }
};
```

## Common Use Cases

### 1. Object Counting

```cpp
class Entity {
    static int totalEntities;

public:
    Entity() { ++totalEntities; }
    ~Entity() { --totalEntities; }

    static int getCount() { return totalEntities; }
};

int Entity::totalEntities = 0;
```

### 2. Singleton Pattern

```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}  // Private constructor

public:
    static Singleton& getInstance() {
        static Singleton instance;  // Thread-safe in C++11
        return instance;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};
```

### 3. Factory Methods

```cpp
class Shape {
public:
    static std::unique_ptr<Shape> create(const std::string& type) {
        if (type == "circle") return std::make_unique<Circle>();
        if (type == "square") return std::make_unique<Square>();
        return nullptr;
    }
};

auto shape = Shape::create("circle");
```

### 4. Constants

```cpp
class HTTPStatus {
public:
    static constexpr int OK = 200;
    static constexpr int NOT_FOUND = 404;
    static constexpr int SERVER_ERROR = 500;
};

if (response.code == HTTPStatus::OK) { /* ... */ }
```

### 5. Caching/Memoization

```cpp
class Fibonacci {
    static std::unordered_map<int, long long> cache;

public:
    static long long compute(int n) {
        if (n <= 1) return n;
        if (cache.count(n)) return cache[n];
        return cache[n] = compute(n-1) + compute(n-2);
    }
};

std::unordered_map<int, long long> Fibonacci::cache;
```

## Static in Templates

Each template instantiation gets its own static variable:

```cpp
template<typename T>
class Container {
public:
    static int count;
};

template<typename T>
int Container<T>::count = 0;

int main() {
    Container<int>::count = 5;
    Container<double>::count = 10;

    // These are DIFFERENT variables!
    std::cout << Container<int>::count;     // 5
    std::cout << Container<double>::count;  // 10
}
```

## Static Local Variables

```cpp
void foo() {
    static int callCount = 0;  // Initialized once, persists
    ++callCount;
    std::cout << "Called " << callCount << " times\n";
}

foo();  // Called 1 times
foo();  // Called 2 times
foo();  // Called 3 times
```

## Thread Safety

```cpp
// C++11: Static local initialization is thread-safe
Singleton& Singleton::getInstance() {
    static Singleton instance;  // Thread-safe initialization
    return instance;
}

// Static member variables are NOT automatically thread-safe
class Counter {
    static int count;  // Needs mutex for thread safety
    static std::mutex mtx;

public:
    static void increment() {
        std::lock_guard<std::mutex> lock(mtx);
        ++count;
    }
};
```

## Key Takeaways

- Static members belong to the class, not instances
- Static variables: one copy shared by all objects
- Static functions: no `this` pointer, can only access static members
- Use `inline static` (C++17) to define in header
- Static local variables are initialized once and persist
- Each template instantiation has separate static members

## Common Interview Questions

> [!question]- Can static member functions access non-static members?
> No. Static functions have no `this` pointer, so they can't access instance members. They can only access static members.

> [!question]- Where do you define static member variables?
> In a .cpp file (or use `inline static` in C++17 headers). The class declaration only declares them.

> [!question]- Is static local initialization thread-safe?
> Yes, in C++11 and later. The compiler ensures only one thread initializes a static local variable.

> [!question]- Do all template instantiations share static members?
> No! Each instantiation (e.g., `Container<int>` and `Container<double>`) has its own separate static members.

## Related Topics

- [[../../Advanced/Design_Patterns/01_singleton|Singleton Pattern]]
- [[03_friend_functions_and_classes|Friend Functions]]
