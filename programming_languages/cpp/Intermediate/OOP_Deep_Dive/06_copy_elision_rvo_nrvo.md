# Copy Elision (RVO/NRVO)

**Copy elision** is a compiler optimization that eliminates unnecessary copy/move operations. Understanding it helps you write efficient code and reason about object lifetimes.

## What Is Copy Elision?

Copy elision allows the compiler to **skip** copying/moving objects, constructing them directly at their final location.

```cpp
class Widget {
public:
    Widget() { std::cout << "Default constructor\n"; }
    Widget(const Widget&) { std::cout << "Copy constructor\n"; }
    Widget(Widget&&) { std::cout << "Move constructor\n"; }
};

Widget createWidget() {
    return Widget();  // Might expect: construct + move
}

int main() {
    Widget w = createWidget();  // Might expect: another copy/move
}
```

**Without elision:** Default constructor → Move → Move (or copy)
**With elision:** Default constructor only!

## Types of Copy Elision

### 1. Return Value Optimization (RVO)

When returning a temporary (prvalue):

```cpp
Widget createWidget() {
    return Widget();  // RVO applies here
}

Widget w = createWidget();  // Direct construction, no copy/move
```

RVO is **mandatory in C++17** for this case (guaranteed copy elision).

### 2. Named Return Value Optimization (NRVO)

When returning a named local variable:

```cpp
Widget createWidget() {
    Widget w;      // Named variable
    // ... modify w ...
    return w;      // NRVO might apply
}
```

NRVO is **not guaranteed**—it's an optional optimization.

### 3. Temporary Materialization (C++17)

When binding a reference to a temporary:

```cpp
const Widget& ref = Widget();  // Temporary constructed in place
```

## RVO vs NRVO

| Aspect | RVO | NRVO |
|--------|-----|------|
| What | Return temporary | Return named variable |
| Guaranteed (C++17)? | Yes | No |
| When it fails | Never (C++17+) | Multiple return paths, etc. |

## When NRVO Might Not Apply

### Multiple Return Paths

```cpp
Widget create(bool flag) {
    Widget a, b;
    if (flag)
        return a;  // Which one to elide?
    else
        return b;  // Can't elide both
}
// Compiler might fall back to move
```

### Returning Parameter

```cpp
Widget process(Widget w) {
    return w;  // NRVO doesn't apply to parameters
}
// Parameter is moved (not elided)
```

### Returning Member

```cpp
class Container {
    Widget member;
public:
    Widget getMember() { return member; }  // Copies, no elision
};
```

## Code Example: Observing Elision

```cpp
#include <iostream>

class Verbose {
public:
    Verbose() { std::cout << "Default ctor\n"; }
    Verbose(const Verbose&) { std::cout << "Copy ctor\n"; }
    Verbose(Verbose&&) noexcept { std::cout << "Move ctor\n"; }
    ~Verbose() { std::cout << "Destructor\n"; }
};

// RVO - guaranteed in C++17
Verbose rvo() {
    return Verbose();
}

// NRVO - might be elided
Verbose nrvo() {
    Verbose v;
    return v;
}

// No elision - multiple return paths
Verbose noElision(bool flag) {
    Verbose a, b;
    return flag ? a : b;
}

int main() {
    std::cout << "=== RVO ===\n";
    Verbose v1 = rvo();

    std::cout << "\n=== NRVO ===\n";
    Verbose v2 = nrvo();

    std::cout << "\n=== No elision ===\n";
    Verbose v3 = noElision(true);

    std::cout << "\n=== Cleanup ===\n";
    return 0;
}
```

Typical output (with elision enabled):
```
=== RVO ===
Default ctor

=== NRVO ===
Default ctor

=== No elision ===
Default ctor
Default ctor
Move ctor
Destructor
Destructor

=== Cleanup ===
Destructor
Destructor
Destructor
```

## Guaranteed Copy Elision (C++17)

C++17 **mandates** copy elision in certain cases:

```cpp
// These MUST be elided in C++17:

// 1. Returning a prvalue of the same type
Widget f() { return Widget(); }
Widget w = f();  // Guaranteed: one construction

// 2. Initializing from a prvalue
Widget w = Widget();  // Guaranteed: one construction

// 3. Throwing/catching prvalues
throw Widget();  // Object constructed directly in exception storage
```

This means you can return non-copyable, non-movable types:

```cpp
class Unique {
public:
    Unique() = default;
    Unique(const Unique&) = delete;
    Unique(Unique&&) = delete;
};

Unique create() {
    return Unique();  // OK in C++17! Guaranteed elision
}

Unique u = create();  // Works!
```

## Impact on Code Design

### Don't Pessimize

```cpp
// BAD - prevents NRVO
Widget create() {
    Widget w;
    return std::move(w);  // Don't do this! Prevents NRVO
}

// GOOD - allows NRVO
Widget create() {
    Widget w;
    return w;  // Let compiler optimize
}
```

### Trust the Compiler

```cpp
// Modern idiom - return by value
std::vector<int> getVector() {
    std::vector<int> result;
    // ... fill result ...
    return result;  // NRVO or move - both efficient
}

// Old idiom - output parameter (unnecessary)
void getVector(std::vector<int>& out) {
    // ... fill out ...
}
```

## Disabling Copy Elision (For Testing)

Compilers allow disabling elision for debugging:

```bash
# GCC/Clang
g++ -fno-elide-constructors program.cpp

# This shows what would happen without optimization
```

## Factory Functions and Elision

Copy elision makes factory functions efficient:

```cpp
class Widget {
public:
    static Widget create(int config) {
        Widget w;
        // configure based on config
        return w;  // NRVO
    }
};

Widget w = Widget::create(42);  // Efficient - no extra copies
```

## Key Takeaways

- Copy elision skips copy/move operations entirely
- RVO: returning temporaries (guaranteed in C++17)
- NRVO: returning named variables (optional optimization)
- Don't use `std::move` on return statements (prevents NRVO)
- Return by value is efficient—trust the compiler
- C++17 guarantees elision for prvalues (enables returning non-movable types)

## Common Interview Questions

> [!question]- What's the difference between RVO and NRVO?
> RVO applies to returning temporaries (prvalues) and is guaranteed in C++17. NRVO applies to returning named local variables and is optional.

> [!question]- Why shouldn't you write `return std::move(local)`?
> It prevents NRVO! Without `std::move`, the compiler can elide the copy entirely. With it, you force a move operation.

> [!question]- Can you return a non-movable type by value?
> In C++17, yes! Guaranteed copy elision means the object is constructed directly in the caller's storage without copying or moving.

> [!question]- Does copy elision affect performance?
> Yes, significantly for large objects. It eliminates potentially expensive copy/move operations entirely.

## Related Topics

- [[05_shallow_vs_deep_copy|Shallow vs Deep Copy]]
- [[../../Advanced/Move_Semantics_and_Rvalue_References/01_lvalues_and_rvalues|Lvalues and Rvalues]]
- [[../../Advanced/Move_Semantics_and_Rvalue_References/03_move_constructors_and_move_assignment|Move Constructors]]
