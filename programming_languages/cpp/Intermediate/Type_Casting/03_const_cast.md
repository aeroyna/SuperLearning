# const_cast

`const_cast` is used to **add or remove** `const` (or `volatile`) qualifiers from a variable. It's the only cast that can do this.

## Syntax

```cpp
const_cast<target_type>(expression)
```

## Primary Use Cases

### 1. Removing const to Call Non-const Functions

Sometimes you have a const object but need to call a function that wasn't designed to take const (often legacy code).

```cpp
#include <iostream>

// Legacy function that doesn't take const (but doesn't modify)
void legacyPrint(char* str) {
    std::cout << str << std::endl;
}

int main() {
    const char* msg = "Hello, World!";

    // legacyPrint(msg);  // Error: can't convert const char* to char*

    legacyPrint(const_cast<char*>(msg));  // OK, if legacyPrint doesn't modify

    return 0;
}
```

> [!warning] Danger
> If `legacyPrint` actually modified `msg`, this would be **undefined behavior**.

### 2. Implementing const and non-const Overloads

Avoid code duplication in getter functions:

```cpp
class Container {
private:
    int* data;
    size_t size;

public:
    // Non-const version
    int& at(size_t index) {
        // Reuse const version to avoid duplication
        return const_cast<int&>(
            static_cast<const Container&>(*this).at(index)
        );
    }

    // Const version (the "real" implementation)
    const int& at(size_t index) const {
        if (index >= size) throw std::out_of_range("Index out of bounds");
        return data[index];
    }
};
```

### 3. Working with APIs That Expect Non-const

```cpp
// Some C APIs don't use const properly
extern "C" void process_buffer(char* buffer, size_t len);  // Doesn't actually modify

void processData(const std::string& str) {
    // Safe ONLY if we're certain process_buffer doesn't modify
    process_buffer(const_cast<char*>(str.c_str()), str.size());
}
```

## Undefined Behavior Warning

**Modifying a truly const object through const_cast is undefined behavior!**

```cpp
const int original = 42;
int* ptr = const_cast<int*>(&original);
*ptr = 100;  // UNDEFINED BEHAVIOR!
// original might still be 42 (compiler optimization)
// or might crash, or might be 100
```

The compiler may place `const` variables in read-only memory or optimize based on const-ness.

## When It's Safe

Casting away const is safe when:
1. The original object was NOT declared const
2. You're just interfacing with APIs that forgot const

```cpp
int mutableValue = 42;

// This is safe - original object is not const
const int* cptr = &mutableValue;
int* ptr = const_cast<int*>(cptr);
*ptr = 100;  // OK! Original object was mutable

std::cout << mutableValue << std::endl;  // 100
```

## Adding const

Less common, but `const_cast` can also ADD const:

```cpp
int* mutablePtr = new int(42);
const int* constPtr = const_cast<const int*>(mutablePtr);

// Though implicit conversion also works:
const int* constPtr2 = mutablePtr;  // Implicit, no cast needed
```

## volatile

`const_cast` also works with `volatile`:

```cpp
volatile int sensor = 0;
int* nonVolatile = const_cast<int*>(&sensor);
// Usually a bad idea - volatile is there for a reason
```

## Alternatives to const_cast

### Use mutable for Class Members

```cpp
class Cache {
private:
    mutable std::map<int, int> cache;  // Can modify in const methods

public:
    int compute(int x) const {
        if (cache.count(x)) return cache[x];
        int result = /* expensive computation */;
        cache[x] = result;  // OK! cache is mutable
        return result;
    }
};
```

### Design Better Interfaces

```cpp
// Instead of casting away const:
void process(char* str);  // Bad: should be const char*

// Design with const:
void process(const char* str);  // Good: expresses intent
```

## Summary Table

| Scenario | Safe? | Example |
|----------|-------|---------|
| Cast away const on originally mutable object | ✅ Yes | `int x; const int* cp = &x; int* p = const_cast<int*>(cp);` |
| Modify truly const object | ❌ UB | `const int x = 5; *const_cast<int*>(&x) = 10;` |
| Interface with legacy non-const API (no modification) | ⚠️ Risky | `legacyFunc(const_cast<char*>(str))` |
| Implement non-const using const version | ✅ Yes | Scott Meyers pattern |

## Key Takeaways

- Only cast that can add/remove const or volatile
- Modifying truly const objects is **undefined behavior**
- Safe when original object wasn't const
- Common use: implement non-const version using const version
- If you use it often, consider improving your design
- `mutable` is often a better solution for class members

## Common Interview Questions

> [!question]- Why is modifying a const object through const_cast UB?
> The compiler may place const objects in read-only memory or optimize code assuming they never change. Modifying violates these assumptions.

> [!question]- When is const_cast actually safe?
> When the original object was not declared const, and you're just working around an API that doesn't properly use const.

> [!question]- What's the alternative to const_cast for caching in const methods?
> Use the `mutable` keyword on the cache member variable.

## Related Topics

- [[../OOP_Fundamentals/04_const_correctness|Const Correctness]]
- [[01_static_cast|static_cast]]
