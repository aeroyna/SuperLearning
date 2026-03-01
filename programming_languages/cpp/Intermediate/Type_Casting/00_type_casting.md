# Type Casting in C++

Type casting (or type conversion) is the process of converting a value from one data type to another. C++ provides several casting mechanisms, each with specific use cases and safety guarantees.

## Why Multiple Cast Types?

C-style casts are powerful but dangerous—they can perform any type of conversion without indicating intent. C++ introduced four named casts that make code safer and more readable.

```cpp
// C-style cast (avoid in C++)
int* p = (int*)malloc(sizeof(int));
double d = (double)5;

// C++ named casts (preferred)
int* p = static_cast<int*>(ptr);
Base* b = dynamic_cast<Base*>(derivedPtr);
```

## The Four C++ Casts

| Cast | Purpose | Checked? |
|------|---------|----------|
| `static_cast` | Compile-time conversions (most common) | Compile-time |
| `dynamic_cast` | Safe downcasting in inheritance | Runtime |
| `const_cast` | Add/remove const qualifier | Compile-time |
| `reinterpret_cast` | Low-level bit reinterpretation | None |

## Quick Comparison

```cpp
class Base { virtual void foo() {} };
class Derived : public Base {};

Base* b = new Derived();

// static_cast: Fast but unsafe for polymorphic downcasting
Derived* d1 = static_cast<Derived*>(b);

// dynamic_cast: Safe, returns nullptr if cast fails
Derived* d2 = dynamic_cast<Derived*>(b);

// const_cast: Remove const (use carefully)
const int x = 10;
int* p = const_cast<int*>(&x);

// reinterpret_cast: Dangerous, raw bit reinterpretation
int* ip = new int(42);
long addr = reinterpret_cast<long>(ip);
```

## Decision Flowchart

```
Need to cast?
├── Converting between related types (int↔double, pointer hierarchy)?
│   └── Use static_cast
├── Downcasting in polymorphic hierarchy safely?
│   └── Use dynamic_cast
├── Adding or removing const/volatile?
│   └── Use const_cast
├── Interpreting bits as different type?
│   └── Use reinterpret_cast (DANGEROUS)
└── None of the above?
    └── Reconsider if cast is necessary
```

## In This Chapter

- [[01_static_cast|static_cast]]
- [[02_dynamic_cast|dynamic_cast]]
- [[03_const_cast|const_cast]]
- [[04_reinterpret_cast|reinterpret_cast]]
- [[05_c_style_vs_cpp_casts|C-Style vs C++ Casts]]
- [[practice_problems|Practice Problems]]

## Key Takeaways

- Avoid C-style casts in C++
- `static_cast` is the most commonly used
- `dynamic_cast` requires polymorphic types (virtual functions)
- `const_cast` should be used sparingly
- `reinterpret_cast` is for low-level programming only
- When in doubt, question whether you need the cast at all

## Common Interview Questions

> [!question]- Why does C++ have multiple cast operators?
> Each cast serves a specific purpose and shows intent. This makes code more readable and allows compilers to catch certain errors. C-style casts hide what kind of conversion is happening.

> [!question]- Which cast is safest?
> `dynamic_cast` because it performs runtime type checking. `static_cast` is safe for its intended uses but can cause UB if misused for downcasting.
