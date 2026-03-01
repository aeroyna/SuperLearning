# Virtual Tables (vtable) and vptr

Understanding how virtual functions work internally is crucial for C++ interviews. The mechanism involves **virtual tables (vtables)** and **virtual pointers (vptrs)**.

## The Problem Virtual Functions Solve

Without virtual functions, the function called is determined at **compile time**:

```cpp
class Base {
public:
    void speak() { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void speak() { std::cout << "Derived\n"; }
};

Base* ptr = new Derived();
ptr->speak();  // Prints "Base" - NOT what we want!
```

With virtual functions, the function is determined at **runtime**:

```cpp
class Base {
public:
    virtual void speak() { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void speak() override { std::cout << "Derived\n"; }
};

Base* ptr = new Derived();
ptr->speak();  // Prints "Derived" - correct!
```

## How It Works: The vtable

When a class has virtual functions, the compiler creates a **virtual table (vtable)** for that class.

### What's in a vtable?

A vtable is an array of function pointers, one for each virtual function:

```
Base's vtable:
┌─────────────────────────────┐
│ ptr to Base::speak()        │
│ ptr to Base::~Base()        │
└─────────────────────────────┘

Derived's vtable:
┌─────────────────────────────┐
│ ptr to Derived::speak()     │  ← Overridden
│ ptr to Derived::~Derived()  │
└─────────────────────────────┘
```

### The vptr (Virtual Pointer)

Each object of a polymorphic class contains a hidden pointer called **vptr** that points to its class's vtable:

```cpp
class Base {
    // Hidden: vptr → Base::vtable
public:
    virtual void speak();
    virtual ~Base();
};

sizeof(Base);  // Includes size of vptr (typically 8 bytes on 64-bit)
```

## Memory Layout Visualization

```cpp
Base* b = new Derived();
```

```
Stack:                    Heap:
┌─────────┐              ┌──────────────────┐
│  b ─────┼─────────────→│ vptr ────────────┼──→ Derived's vtable
└─────────┘              │ Base members...  │    ┌─────────────────┐
                         │ Derived members..│    │ Derived::speak  │
                         └──────────────────┘    │ Derived::~Derived│
                                                 └─────────────────┘
```

## Virtual Function Call Mechanism

When you call `b->speak()`:

1. **Dereference** `b` to get the object
2. **Read the vptr** from the object (hidden first member)
3. **Index into vtable** for `speak()` slot
4. **Call the function** through the pointer

```cpp
// Conceptually equivalent to:
(*(b->vptr[0]))();  // Index 0 for speak()
```

This is why virtual calls have slight overhead compared to non-virtual calls.

## Code Example: Observing vtable Behavior

```cpp
#include <iostream>

class Animal {
public:
    virtual void speak() { std::cout << "Animal speaks\n"; }
    virtual void eat() { std::cout << "Animal eats\n"; }
    virtual ~Animal() = default;
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "Woof!\n"; }
    // eat() is not overridden - uses Animal::eat
};

class Cat : public Animal {
public:
    void speak() override { std::cout << "Meow!\n"; }
    void eat() override { std::cout << "Cat eats fish\n"; }
};

int main() {
    Animal* animals[] = { new Dog(), new Cat(), new Animal() };

    for (Animal* a : animals) {
        a->speak();  // Virtual dispatch
        a->eat();    // Virtual dispatch
        std::cout << "---\n";
    }

    for (Animal* a : animals) delete a;
    return 0;
}
```

Output:
```
Woof!
Animal eats
---
Meow!
Cat eats fish
---
Animal speaks
Animal eats
---
```

## Performance Implications

### Virtual Call Overhead

1. **Extra indirection**: Two pointer dereferences (vptr → vtable → function)
2. **Cache miss potential**: vtable might not be in cache
3. **Cannot be inlined**: Compiler can't inline unknown function

### Size Overhead

```cpp
class NonVirtual {
    int x;  // 4 bytes
};
// sizeof(NonVirtual) = 4

class Virtual {
    virtual void foo() {}
    int x;  // 4 bytes
};
// sizeof(Virtual) = 16 (8 byte vptr + 4 byte int + padding)
```

### When Compiler Can Optimize

```cpp
Derived d;
d.speak();  // Compiler KNOWS exact type - might devirtualize

Derived* dp = new Derived();
dp->speak();  // Compiler might devirtualize if it can prove type
```

## Multiple Inheritance and vtables

With multiple inheritance, objects may have **multiple vptrs**:

```cpp
class A {
    virtual void foo();
};

class B {
    virtual void bar();
};

class C : public A, public B {
    void foo() override;
    void bar() override;
};
```

```
C object layout:
┌────────────────┐
│ vptr to A part │ → A-compatible vtable (foo)
│ A members      │
├────────────────┤
│ vptr to B part │ → B-compatible vtable (bar)
│ B members      │
├────────────────┤
│ C members      │
└────────────────┘
```

## Pure Virtual Functions in vtable

Pure virtual functions (`= 0`) have a special entry:

```cpp
class Abstract {
    virtual void pure() = 0;  // vtable entry points to __cxa_pure_virtual
};
```

Calling a pure virtual function causes program termination.

## Debugging vtables

You can examine vtables with compiler flags:

```bash
# GCC/Clang: dump class layout
g++ -fdump-class-hierarchy file.cpp
clang++ -Xclang -fdump-record-layouts file.cpp

# GCC: dump vtable
g++ -fdump-lang-class file.cpp
```

## Key Takeaways

- vtable is a per-class table of function pointers
- vptr is a per-object hidden pointer to the vtable
- Virtual calls have slight overhead (indirection, no inlining)
- Objects with virtual functions are larger (vptr storage)
- Understanding vtables helps debug polymorphism issues

## Common Interview Questions

> [!question]- How much memory overhead does a virtual function add?
> One vptr per object (typically 8 bytes on 64-bit systems). The vtable itself is shared among all instances of a class.

> [!question]- Can the compiler optimize virtual calls?
> Yes, through devirtualization. If the compiler can prove the exact type at compile time, it can call the function directly.

> [!question]- Where is the vtable stored?
> In the program's read-only data section. There's one vtable per polymorphic class, shared by all instances.

> [!question]- Why can't constructors be virtual?
> The vptr is set up during construction. When the constructor runs, the vptr points to the current class's vtable, not any derived class. Virtual dispatch wouldn't work correctly.

> [!question]- Why should destructors be virtual?
> When deleting through a base pointer, the vtable ensures the derived destructor is called first, preventing resource leaks.

## Related Topics

- [[../Object-Oriented_Programming/Polymorphism/02_virtual_functions|Virtual Functions]]
- [[02_diamond_problem|Diamond Problem]]
- [[../Type_Casting/02_dynamic_cast|dynamic_cast (uses RTTI from vtable)]]
