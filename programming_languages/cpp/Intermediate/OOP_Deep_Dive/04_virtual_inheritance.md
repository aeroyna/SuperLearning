# Virtual Inheritance

Virtual inheritance is C++'s solution to the diamond problem. This page covers the details and implementation considerations.

## Quick Recap: The Problem

```cpp
class A { public: int data; };
class B : public A { };
class C : public A { };
class D : public B, public C { };  // D has TWO copies of A!
```

## The Solution: Virtual Inheritance

```cpp
class A { public: int data; };
class B : virtual public A { };
class C : virtual public A { };
class D : public B, public C { };  // D has ONE copy of A
```

## Syntax

```cpp
class Derived : virtual public Base { };
class Derived : public virtual Base { };  // Same thing, order doesn't matter
```

## How Virtual Inheritance Works Internally

### Without Virtual Inheritance

```
D object layout:
┌─────────┐
│ A (B's) │ ← First copy of A
│ B data  │
├─────────┤
│ A (C's) │ ← Second copy of A
│ C data  │
├─────────┤
│ D data  │
└─────────┘
```

### With Virtual Inheritance

```
D object layout:
┌─────────────┐
│ B's vbptr ──┼──┐
│ B data      │  │
├─────────────┤  │
│ C's vbptr ──┼──┤
│ C data      │  │
├─────────────┤  │
│ D data      │  │
├─────────────┤  │
│ A (shared) ←┼──┘  Single shared copy
│ A data      │
└─────────────┘
```

The **vbptr** (virtual base pointer) in each virtually-inheriting subobject points to the shared base.

## Constructor Initialization Rules

**The most derived class must initialize the virtual base:**

```cpp
class A {
public:
    A(int x) : value(x) { std::cout << "A(" << x << ")\n"; }
    int value;
};

class B : virtual public A {
public:
    B(int x) : A(x) { std::cout << "B\n"; }  // A(x) ignored if B is not most derived
};

class C : virtual public A {
public:
    C(int x) : A(x) { std::cout << "C\n"; }  // A(x) ignored if C is not most derived
};

class D : public B, public C {
public:
    // D MUST initialize A directly
    D(int x) : A(x), B(x), C(x) { std::cout << "D\n"; }
};

int main() {
    D d(42);
    return 0;
}
```

Output:
```
A(42)
B
C
D
```

Note: `A` is constructed **once**, and by `D`, not by `B` or `C`.

## Construction Order

With virtual inheritance, the order is:

1. **Virtual bases** (in declaration order, depth-first, left-to-right)
2. **Non-virtual bases** (in declaration order)
3. **Members** (in declaration order)
4. **Constructor body**

```cpp
class V1 { V1() { std::cout << "V1 "; } };
class V2 { V2() { std::cout << "V2 "; } };
class B1 : virtual public V1 { B1() { std::cout << "B1 "; } };
class B2 : virtual public V2 { B2() { std::cout << "B2 "; } };
class D : public B1, public B2 {
    D() { std::cout << "D"; }
};

D d;  // Output: V1 V2 B1 B2 D
```

## Accessing the Virtual Base

```cpp
class A { public: int x = 0; };
class B : virtual public A { };
class C : virtual public A { };
class D : public B, public C { };

D d;
d.x = 42;           // OK - unambiguous, only one A::x

B& b = d;
C& c = d;
std::cout << b.x;   // 42
std::cout << c.x;   // 42 (same x!)
```

## Performance Implications

| Aspect | Non-Virtual Inheritance | Virtual Inheritance |
|--------|------------------------|---------------------|
| Object size | Smaller | Larger (vbptr overhead) |
| Base access | Direct offset | Indirect (through vbptr) |
| Construction | Simple | Complex (most-derived responsibility) |
| Conversion to base | Simple pointer adjustment | May need vbptr lookup |

## Practical Example: Interface Hierarchy

A common real-world use is in interface hierarchies:

```cpp
// Common interface
class ISerializable {
public:
    virtual void serialize() = 0;
    virtual ~ISerializable() = default;
};

// Specific interfaces inherit virtually
class IReadable : virtual public ISerializable {
public:
    virtual void read() = 0;
};

class IWritable : virtual public ISerializable {
public:
    virtual void write() = 0;
};

// Concrete class implements all
class File : public IReadable, public IWritable {
public:
    void serialize() override { /* ... */ }
    void read() override { /* ... */ }
    void write() override { /* ... */ }
};

// Only ONE ISerializable in File
```

## Common Pitfalls

### 1. Forgetting to Initialize Virtual Base

```cpp
class D : public B, public C {
    D() : B(1), C(2) { }  // ERROR: A has no default constructor
    // Must add: A(something)
};
```

### 2. Default Arguments in Virtual Base

```cpp
class A {
public:
    A(int x = 0) : value(x) { }
    int value;
};

class B : virtual public A { };
class C : virtual public A { };

class D : public B, public C {
    D() { }  // OK - uses A's default constructor
};

D d;
std::cout << d.value;  // 0
```

### 3. Mixing Virtual and Non-Virtual

```cpp
class A { };
class B : virtual public A { };
class C : public A { };  // NOT virtual!
class D : public B, public C { };  // D has TWO A's!

// B's A and C's A are different!
```

## Casting with Virtual Inheritance

```cpp
class A { };
class B : virtual public A { };

B* b = new B();

// Upcast works normally
A* a = b;  // OK

// Downcast needs dynamic_cast for virtual bases
// static_cast doesn't work because offset isn't fixed
B* b2 = dynamic_cast<B*>(a);  // OK
// B* b3 = static_cast<B*>(a);  // Error!
```

## When to Use Virtual Inheritance

**Use when:**
- You have a true diamond pattern
- Shared base semantics make sense
- Building interface hierarchies

**Avoid when:**
- No diamond pattern exists
- You can redesign to avoid it
- Performance is critical

## Alternatives

### Composition

```cpp
class FlyingMammal {
    MammalBehavior mammal;
    BirdBehavior bird;
    // Delegate to members
};
```

### Traits/Mixins with CRTP

```cpp
template<typename Derived>
class Printable {
public:
    void print() {
        static_cast<Derived*>(this)->printImpl();
    }
};
```

## Key Takeaways

- Virtual inheritance creates single shared instance of base
- Most derived class initializes virtual bases
- vbptr enables finding the shared base at runtime
- Has memory and performance overhead
- Essential for interface hierarchies
- Cannot use `static_cast` to downcast from virtual base

## Common Interview Questions

> [!question]- Why can't you use static_cast with virtual inheritance?
> The offset from the virtual base to the derived class isn't known at compile time—it depends on the most derived class. Only `dynamic_cast` can compute it at runtime.

> [!question]- What's the construction order with virtual inheritance?
> Virtual bases are constructed first (by the most derived class), then non-virtual bases, then the most derived class itself.

> [!question]- Does virtual inheritance affect performance?
> Yes. Accessing the virtual base requires following the vbptr (extra indirection). Object size increases due to vbptr storage.

## Related Topics

- [[02_diamond_problem|Diamond Problem]]
- [[01_vtable_and_vptr|vtable and vptr]]
- [[../Type_Casting/02_dynamic_cast|dynamic_cast]]
