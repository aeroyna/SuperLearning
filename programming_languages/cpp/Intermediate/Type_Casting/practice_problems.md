# Practice Problems: Type Casting

## Problem 1: Cast Identification ⭐ (Easy)

For each C-style cast below, identify which C++ cast(s) it's equivalent to:

```cpp
int x = 10;
const int* cp = &x;
void* vp = &x;
class Base { virtual ~Base() {} };
class Derived : public Base {};
Base* bp = new Derived();

// 1. double d = (double)x;
// 2. int* p = (int*)cp;
// 3. int* p = (int*)vp;
// 4. Derived* dp = (Derived*)bp;
// 5. long addr = (long)&x;
```

> [!success]- Solution
> 1. `static_cast<double>(x)` - numeric conversion
> 2. `const_cast<int*>(cp)` - removing const
> 3. `static_cast<int*>(vp)` - void pointer conversion
> 4. `static_cast<Derived*>(bp)` (unsafe) or `dynamic_cast<Derived*>(bp)` (safe)
> 5. `reinterpret_cast<long>(&x)` - pointer to integer

---

## Problem 2: Safe Downcast ⭐⭐ (Medium)

You have a vector of `Animal*` pointers. Some are `Dog*`, some are `Cat*`. Write a function that counts how many dogs are in the vector.

```cpp
class Animal {
public:
    virtual ~Animal() = default;
    virtual void speak() = 0;
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "Woof!\n"; }
    void fetch() { std::cout << "Fetching ball!\n"; }
};

class Cat : public Animal {
public:
    void speak() override { std::cout << "Meow!\n"; }
};
```

> [!hint]- Hint
> Use `dynamic_cast` and check for `nullptr`.

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <vector>
>
> // ... Animal, Dog, Cat classes ...
>
> int countDogs(const std::vector<Animal*>& animals) {
>     int count = 0;
>     for (Animal* animal : animals) {
>         if (dynamic_cast<Dog*>(animal) != nullptr) {
>             ++count;
>         }
>     }
>     return count;
> }
>
> // Alternative using algorithm
> int countDogs2(const std::vector<Animal*>& animals) {
>     return std::count_if(animals.begin(), animals.end(),
>         [](Animal* a) { return dynamic_cast<Dog*>(a) != nullptr; });
> }
>
> // Make all dogs fetch
> void makeAllDogsFetch(const std::vector<Animal*>& animals) {
>     for (Animal* animal : animals) {
>         if (Dog* dog = dynamic_cast<Dog*>(animal)) {
>             dog->fetch();
>         }
>     }
> }
>
> int main() {
>     std::vector<Animal*> animals = {
>         new Dog(), new Cat(), new Dog(), new Cat(), new Dog()
>     };
>
>     std::cout << "Dogs: " << countDogs(animals) << std::endl;  // 3
>
>     makeAllDogsFetch(animals);  // 3x "Fetching ball!"
>
>     for (Animal* a : animals) delete a;
>     return 0;
> }
> ```

---

## Problem 3: Avoiding const_cast ⭐⭐ (Medium)

Refactor this code to eliminate the `const_cast`:

```cpp
class Cache {
private:
    std::map<int, int> cache;

public:
    int compute(int x) const {
        if (cache.count(x)) {
            return cache.at(x);
        }
        int result = x * x;  // Expensive computation
        const_cast<Cache*>(this)->cache[x] = result;  // Bad!
        return result;
    }
};
```

> [!success]- Solution
> ```cpp
> #include <map>
>
> class Cache {
> private:
>     mutable std::map<int, int> cache;  // mutable allows modification in const methods
>
> public:
>     int compute(int x) const {
>         if (cache.count(x)) {
>             return cache.at(x);
>         }
>         int result = x * x;
>         cache[x] = result;  // OK! cache is mutable
>         return result;
>     }
> };
>
> // Alternative: Make compute non-const if caching is an observable side effect
> class Cache2 {
> private:
>     std::map<int, int> cache;
>
> public:
>     int compute(int x) {  // Non-const
>         if (cache.count(x)) {
>             return cache[x];
>         }
>         int result = x * x;
>         cache[x] = result;
>         return result;
>     }
> };
> ```

---

## Problem 4: Serialization ⭐⭐⭐ (Hard)

Write functions to serialize and deserialize a `float` to/from a byte array using proper casting techniques.

> [!hint]- Hint
> Use `memcpy` or `std::bit_cast` (C++20) for safe type punning, NOT `reinterpret_cast`.

> [!success]- Solution
> ```cpp
> #include <iostream>
> #include <cstring>
> #include <array>
>
> // C++20 version with std::bit_cast
> #if __cplusplus >= 202002L
> #include <bit>
>
> std::array<std::byte, sizeof(float)> serializeFloat(float f) {
>     auto bits = std::bit_cast<std::array<std::byte, sizeof(float)>>(f);
>     return bits;
> }
>
> float deserializeFloat(const std::array<std::byte, sizeof(float)>& bytes) {
>     return std::bit_cast<float>(bytes);
> }
> #endif
>
> // Pre-C++20 version with memcpy (portable and safe)
> void serializeFloatToBytes(float f, unsigned char* out) {
>     std::memcpy(out, &f, sizeof(float));
> }
>
> float deserializeFloatFromBytes(const unsigned char* bytes) {
>     float result;
>     std::memcpy(&result, bytes, sizeof(float));
>     return result;
> }
>
> // Printing bytes helper
> void printBytes(const unsigned char* bytes, size_t n) {
>     std::cout << std::hex;
>     for (size_t i = 0; i < n; ++i) {
>         std::cout << static_cast<int>(bytes[i]) << " ";
>     }
>     std::cout << std::dec << std::endl;
> }
>
> int main() {
>     float original = 3.14159f;
>     unsigned char buffer[sizeof(float)];
>
>     serializeFloatToBytes(original, buffer);
>     std::cout << "Bytes: ";
>     printBytes(buffer, sizeof(float));
>
>     float restored = deserializeFloatFromBytes(buffer);
>     std::cout << "Restored: " << restored << std::endl;
>
>     std::cout << "Match: " << (original == restored ? "Yes" : "No") << std::endl;
>
>     return 0;
> }
> ```

---

## Problem 5: Which Cast? ⭐ (Easy)

For each scenario, identify the correct C++ cast:

1. Converting `int` to `double`
2. Downcasting `Base*` to `Derived*` with runtime safety
3. Removing `const` from `const char*`
4. Converting a pointer to `uintptr_t`
5. Upcasting `Derived*` to `Base*`
6. Converting `void*` back to `int*`
7. Converting `enum class Color` to `int`

> [!success]- Solution
> 1. `static_cast<double>(intValue)` - numeric conversion
> 2. `dynamic_cast<Derived*>(basePtr)` - safe downcast
> 3. `const_cast<char*>(constCharPtr)` - remove const
> 4. `reinterpret_cast<uintptr_t>(ptr)` - pointer to integer
> 5. No cast needed (implicit) or `static_cast` if explicit
> 6. `static_cast<int*>(voidPtr)` - void* conversion
> 7. `static_cast<int>(colorValue)` - enum to int

---

## Problem 6: Fix the Bugs ⭐⭐ (Medium)

This code has casting-related bugs. Find and fix them:

```cpp
class Shape { public: virtual ~Shape() {} };
class Circle : public Shape { public: double radius; };
class Square : public Shape { public: double side; };

void processShape(const Shape* shape) {
    // Bug 1: Using static_cast for runtime polymorphism
    Circle* c = static_cast<Circle*>(shape);
    if (c) {
        std::cout << "Circle radius: " << c->radius << std::endl;
    }

    // Bug 2: Wrong cast for const removal
    Shape* mutableShape = static_cast<Shape*>(shape);
}

void examineMemory(int* ptr) {
    // Bug 3: Strict aliasing violation
    float* fp = reinterpret_cast<float*>(ptr);
    std::cout << *fp << std::endl;
}
```

> [!success]- Solution
> ```cpp
> class Shape { public: virtual ~Shape() {} };
> class Circle : public Shape { public: double radius; };
> class Square : public Shape { public: double side; };
>
> void processShape(const Shape* shape) {
>     // Fix 1: Use dynamic_cast for safe runtime downcasting
>     const Circle* c = dynamic_cast<const Circle*>(shape);
>     if (c) {
>         std::cout << "Circle radius: " << c->radius << std::endl;
>     }
>
>     // Fix 2: Use const_cast to remove const (if necessary)
>     Shape* mutableShape = const_cast<Shape*>(shape);
>     // But question: WHY do we need to remove const? Often a design smell.
> }
>
> void examineMemory(int* ptr) {
>     // Fix 3: Use memcpy for safe type punning
>     float f;
>     std::memcpy(&f, ptr, sizeof(float));
>     std::cout << f << std::endl;
>
>     // Or examine raw bytes (allowed to alias through char*)
>     unsigned char* bytes = reinterpret_cast<unsigned char*>(ptr);
>     for (size_t i = 0; i < sizeof(int); ++i) {
>         std::cout << std::hex << static_cast<int>(bytes[i]) << " ";
>     }
> }
> ```

---

## Problem 7: Interview Question ⭐⭐⭐ (Hard)

Explain what happens in this code and why:

```cpp
const int x = 42;
int* p = const_cast<int*>(&x);
*p = 100;
std::cout << "x = " << x << std::endl;
std::cout << "*p = " << *p << std::endl;
std::cout << "&x = " << &x << std::endl;
std::cout << "p = " << p << std::endl;
```

> [!success]- Solution
> **This is undefined behavior!**
>
> Possible outputs (implementation-dependent):
> ```
> x = 42     // Compiler optimized, using original const value
> *p = 100   // Memory was actually modified
> &x = 0x7fff...
> p = 0x7fff...  // Same address!
> ```
>
> **Why this happens:**
> 1. `x` is declared `const`, so the compiler may:
>    - Place it in read-only memory (would crash on write)
>    - Substitute `42` directly wherever `x` is used (constant folding)
>
> 2. `const_cast` removes the const qualifier but doesn't change the fact that `x` was originally declared const
>
> 3. Modifying a truly const object is UB—anything can happen:
>    - The modification might succeed
>    - The program might crash
>    - The original value might still appear due to optimization
>
> **Key lesson:** `const_cast` is only safe when the original object was NOT declared const.
>
> ```cpp
> // SAFE:
> int y = 42;  // Not const
> const int* cp = &y;
> int* p = const_cast<int*>(cp);
> *p = 100;  // OK! Original object was mutable
>
> // UNSAFE (UB):
> const int x = 42;  // const
> int* p = const_cast<int*>(&x);
> *p = 100;  // UB!
> ```
