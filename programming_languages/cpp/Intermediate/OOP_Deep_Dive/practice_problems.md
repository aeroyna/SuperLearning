# Practice Problems: OOP Deep Dive

## Problem 1: vtable Understanding ⭐ (Easy)

What is the output of the following code?

```cpp
#include <iostream>

class A {
public:
    virtual void foo() { std::cout << "A::foo "; }
    void bar() { std::cout << "A::bar "; }
};

class B : public A {
public:
    void foo() override { std::cout << "B::foo "; }
    void bar() { std::cout << "B::bar "; }
};

int main() {
    A* ptr = new B();
    ptr->foo();
    ptr->bar();
    delete ptr;
    return 0;
}
```

> [!success]- Solution
> Output: `B::foo A::bar `
>
> Explanation:
> - `foo()` is virtual, so virtual dispatch occurs. The vtable lookup finds `B::foo`.
> - `bar()` is NOT virtual, so it's resolved at compile time based on pointer type (`A*`), calling `A::bar`.

---

## Problem 2: Diamond Problem Fix ⭐⭐ (Medium)

Fix this code to eliminate the diamond problem:

```cpp
class Device {
public:
    std::string name;
    Device(const std::string& n) : name(n) {}
};

class USBDevice : public Device {
public:
    USBDevice(const std::string& n) : Device(n) {}
};

class NetworkDevice : public Device {
public:
    NetworkDevice(const std::string& n) : Device(n) {}
};

class USBNetworkAdapter : public USBDevice, public NetworkDevice {
public:
    USBNetworkAdapter(const std::string& n)
        : USBDevice(n), NetworkDevice(n) {}  // Two Device objects!
};
```

> [!success]- Solution
> ```cpp
> class Device {
> public:
>     std::string name;
>     Device(const std::string& n = "") : name(n) {}
> };
>
> class USBDevice : virtual public Device {  // virtual!
> public:
>     USBDevice(const std::string& n) : Device(n) {}
> };
>
> class NetworkDevice : virtual public Device {  // virtual!
> public:
>     NetworkDevice(const std::string& n) : Device(n) {}
> };
>
> class USBNetworkAdapter : public USBDevice, public NetworkDevice {
> public:
>     USBNetworkAdapter(const std::string& n)
>         : Device(n),  // Must initialize virtual base directly
>           USBDevice(n),
>           NetworkDevice(n) {}
> };
>
> int main() {
>     USBNetworkAdapter adapter("USB-NET-001");
>     std::cout << adapter.name << std::endl;  // No ambiguity!
>     return 0;
> }
> ```

---

## Problem 3: Object Slicing Prevention ⭐⭐ (Medium)

This code has an object slicing bug. Find and fix it:

```cpp
#include <iostream>
#include <vector>

class Animal {
public:
    virtual void speak() const { std::cout << "Animal\n"; }
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Woof!\n"; }
};

class Cat : public Animal {
public:
    void speak() const override { std::cout << "Meow!\n"; }
};

void makeAllSpeak(std::vector<Animal> animals) {
    for (auto& a : animals) {
        a.speak();
    }
}

int main() {
    std::vector<Animal> zoo;
    zoo.push_back(Dog());
    zoo.push_back(Cat());

    makeAllSpeak(zoo);
    return 0;
}
```

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <vector>
> #include <memory>
>
> class Animal {
> public:
>     virtual void speak() const { std::cout << "Animal\n"; }
>     virtual ~Animal() = default;
> };
>
> class Dog : public Animal {
> public:
>     void speak() const override { std::cout << "Woof!\n"; }
> };
>
> class Cat : public Animal {
> public:
>     void speak() const override { std::cout << "Meow!\n"; }
> };
>
> // Use vector of smart pointers
> void makeAllSpeak(const std::vector<std::unique_ptr<Animal>>& animals) {
>     for (const auto& a : animals) {
>         a->speak();
>     }
> }
>
> int main() {
>     std::vector<std::unique_ptr<Animal>> zoo;
>     zoo.push_back(std::make_unique<Dog>());
>     zoo.push_back(std::make_unique<Cat>());
>
>     makeAllSpeak(zoo);  // Output: Woof! Meow!
>     return 0;
> }
> ```

---

## Problem 4: Deep Copy Implementation ⭐⭐ (Medium)

Implement proper deep copy for this `Matrix` class:

```cpp
class Matrix {
private:
    int** data;
    size_t rows, cols;

public:
    Matrix(size_t r, size_t c) : rows(r), cols(c) {
        data = new int*[rows];
        for (size_t i = 0; i < rows; ++i) {
            data[i] = new int[cols]();
        }
    }

    ~Matrix() {
        for (size_t i = 0; i < rows; ++i) {
            delete[] data[i];
        }
        delete[] data;
    }

    // TODO: Implement copy constructor and copy assignment
};
```

> [!success]- Solution
> ```cpp
> class Matrix {
> private:
>     int** data;
>     size_t rows, cols;
>
>     void allocate() {
>         data = new int*[rows];
>         for (size_t i = 0; i < rows; ++i) {
>             data[i] = new int[cols]();
>         }
>     }
>
>     void deallocate() {
>         for (size_t i = 0; i < rows; ++i) {
>             delete[] data[i];
>         }
>         delete[] data;
>     }
>
>     void copyFrom(const Matrix& other) {
>         for (size_t i = 0; i < rows; ++i) {
>             for (size_t j = 0; j < cols; ++j) {
>                 data[i][j] = other.data[i][j];
>             }
>         }
>     }
>
> public:
>     Matrix(size_t r, size_t c) : rows(r), cols(c) {
>         allocate();
>     }
>
>     // Deep copy constructor
>     Matrix(const Matrix& other) : rows(other.rows), cols(other.cols) {
>         allocate();
>         copyFrom(other);
>     }
>
>     // Deep copy assignment (copy-and-swap)
>     Matrix& operator=(Matrix other) {  // Pass by value
>         swap(other);
>         return *this;
>     }
>
>     // Move constructor
>     Matrix(Matrix&& other) noexcept
>         : data(other.data), rows(other.rows), cols(other.cols) {
>         other.data = nullptr;
>         other.rows = other.cols = 0;
>     }
>
>     void swap(Matrix& other) noexcept {
>         std::swap(data, other.data);
>         std::swap(rows, other.rows);
>         std::swap(cols, other.cols);
>     }
>
>     ~Matrix() {
>         if (data) deallocate();
>     }
>
>     int& at(size_t r, size_t c) { return data[r][c]; }
> };
> ```

---

## Problem 5: Predict the Output ⭐⭐⭐ (Hard)

What's the output? Explain the vtable behavior:

```cpp
#include <iostream>

class Base {
public:
    Base() { print(); }
    virtual void print() { std::cout << "Base::print\n"; }
    virtual ~Base() { print(); }
};

class Derived : public Base {
public:
    Derived() { print(); }
    void print() override { std::cout << "Derived::print\n"; }
    ~Derived() { print(); }
};

int main() {
    Derived d;
    return 0;
}
```

> [!success]- Solution
> Output:
> ```
> Base::print
> Derived::print
> Derived::print
> Base::print
> ```
>
> Explanation:
> 1. **Base constructor runs first**: At this point, the vptr points to `Base`'s vtable (Derived doesn't exist yet), so `Base::print` is called.
> 2. **Derived constructor runs**: Now vptr points to `Derived`'s vtable, so `Derived::print` is called.
> 3. **Derived destructor runs**: vptr still points to `Derived`'s vtable, so `Derived::print` is called.
> 4. **Base destructor runs**: vptr has been changed to point to `Base`'s vtable (Derived part is already destroyed), so `Base::print` is called.
>
> **Key insight**: During construction/destruction, the vptr points to the current class's vtable, not the most derived class.

---

## Problem 6: Copy Elision Analysis ⭐⭐ (Medium)

Which functions will benefit from RVO/NRVO?

```cpp
class Widget {
public:
    Widget();
    Widget(const Widget&);
    Widget(Widget&&);
};

// A
Widget createA() {
    return Widget();
}

// B
Widget createB() {
    Widget w;
    return w;
}

// C
Widget createC() {
    Widget w;
    return std::move(w);
}

// D
Widget createD(bool flag) {
    Widget a, b;
    if (flag) return a;
    return b;
}

// E
Widget createE(Widget w) {
    return w;
}
```

> [!success]- Solution
> | Function | Elision Type | Guaranteed (C++17)? | Notes |
> |----------|-------------|---------------------|-------|
> | A | RVO | Yes | Returning prvalue |
> | B | NRVO | No | Returning named local (likely elided) |
> | C | None | N/A | `std::move` prevents NRVO, forces move |
> | D | Unlikely | No | Multiple return paths make NRVO difficult |
> | E | None | N/A | Can't elide parameter, will move |
>
> **Best practices:**
> - A: Perfect, guaranteed elision
> - B: Good, likely elided
> - C: Bad, don't use `std::move` on return
> - D: Acceptable, will move if not elided
> - E: Necessary evil, parameter can't be elided

---

## Problem 7: Virtual Destructor Necessity ⭐⭐ (Medium)

What's wrong with this code? Fix it.

```cpp
class Resource {
    int* data;
public:
    Resource() : data(new int[100]) {
        std::cout << "Resource allocated\n";
    }
    ~Resource() {
        delete[] data;
        std::cout << "Resource freed\n";
    }
};

class ExtendedResource : public Resource {
    int* moreData;
public:
    ExtendedResource() : moreData(new int[200]) {
        std::cout << "Extended allocated\n";
    }
    ~ExtendedResource() {
        delete[] moreData;
        std::cout << "Extended freed\n";
    }
};

int main() {
    Resource* r = new ExtendedResource();
    delete r;  // ???
    return 0;
}
```

> [!success]- Solution
> **Problem**: `~Resource()` is not virtual!
>
> Output (bug):
> ```
> Resource allocated
> Extended allocated
> Resource freed
> ```
> `moreData` is leaked because `~ExtendedResource()` is never called.
>
> **Fixed code:**
> ```cpp
> class Resource {
>     int* data;
> public:
>     Resource() : data(new int[100]) {
>         std::cout << "Resource allocated\n";
>     }
>     virtual ~Resource() {  // VIRTUAL!
>         delete[] data;
>         std::cout << "Resource freed\n";
>     }
> };
>
> // ExtendedResource stays the same
> ```
>
> Output (fixed):
> ```
> Resource allocated
> Extended allocated
> Extended freed
> Resource freed
> ```
>
> **Rule**: If a class has virtual functions, its destructor should be virtual.

---

## Problem 8: Interview Question ⭐⭐⭐ (Hard)

Explain what happens and why:

```cpp
class Base {
public:
    virtual void foo() { std::cout << "Base::foo\n"; }
};

class Derived : public Base {
public:
    void foo() override { std::cout << "Derived::foo\n"; }
};

int main() {
    Derived d;
    Base* bp = &d;
    Base& br = d;
    Base b = d;  // Slicing!

    bp->foo();  // ?
    br.foo();   // ?
    b.foo();    // ?

    return 0;
}
```

> [!success]- Solution
> Output:
> ```
> Derived::foo
> Derived::foo
> Base::foo
> ```
>
> Explanation:
> - `bp->foo()`: Pointer to Derived object. Virtual dispatch uses Derived's vtable → `Derived::foo`
> - `br.foo()`: Reference to Derived object. Same as pointer → `Derived::foo`
> - `b.foo()`: **Object slicing occurred!** `b` is a `Base` object (not `Derived`), with `Base`'s vtable → `Base::foo`
>
> Key insight: Object slicing copies only the Base portion, including changing the vptr to point to Base's vtable.
