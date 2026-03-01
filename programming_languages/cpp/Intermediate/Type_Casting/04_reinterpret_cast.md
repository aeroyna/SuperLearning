# reinterpret_cast

`reinterpret_cast` is the most dangerous cast. It performs **low-level reinterpretation of bit patterns** without any type checking.

## Syntax

```cpp
reinterpret_cast<target_type>(expression)
```

## What It Does

It tells the compiler: "Trust me, treat these bits as a different type."

```cpp
int x = 42;
int* ip = &x;

// Treat pointer as an integer (memory address)
uintptr_t addr = reinterpret_cast<uintptr_t>(ip);
std::cout << "Address: " << std::hex << addr << std::endl;

// Convert back
int* ip2 = reinterpret_cast<int*>(addr);
std::cout << *ip2 << std::endl;  // 42
```

## Valid Use Cases

### 1. Pointer to Integer and Back

For serialization, debugging, or interfacing with hardware:

```cpp
#include <cstdint>

void* ptr = malloc(100);
uintptr_t address = reinterpret_cast<uintptr_t>(ptr);
std::cout << "Allocated at: 0x" << std::hex << address << std::endl;

// Convert back
void* ptr2 = reinterpret_cast<void*>(address);
free(ptr2);
```

### 2. Type Punning (Reading Raw Bytes)

Examining the binary representation of data:

```cpp
float f = 3.14f;
uint32_t* bits = reinterpret_cast<uint32_t*>(&f);
std::cout << "Float bits: 0x" << std::hex << *bits << std::endl;

// WARNING: This is technically UB due to strict aliasing
// Use std::bit_cast (C++20) or memcpy for portable code
```

### 3. Hardware/Memory-Mapped I/O

```cpp
// Memory-mapped hardware register
volatile uint32_t* gpio_register =
    reinterpret_cast<volatile uint32_t*>(0x40020000);

*gpio_register = 0x01;  // Write to hardware register
```

### 4. Interfacing with C APIs

```cpp
// Callback with void* user data
void callback(void* user_data) {
    MyClass* obj = reinterpret_cast<MyClass*>(user_data);
    obj->doSomething();
}

// Registration
MyClass myObj;
register_callback(callback, reinterpret_cast<void*>(&myObj));
```

### 5. Function Pointer Conversions

```cpp
using FuncPtr = void(*)();

void myFunction() {
    std::cout << "Called!" << std::endl;
}

// Store function pointer as data
uintptr_t funcAddr = reinterpret_cast<uintptr_t>(&myFunction);

// Restore and call
FuncPtr restored = reinterpret_cast<FuncPtr>(funcAddr);
restored();  // "Called!"
```

## Strict Aliasing Rule

**Reading an object through a pointer of different type is usually undefined behavior!**

```cpp
int x = 42;
float* fp = reinterpret_cast<float*>(&x);
float f = *fp;  // UB! Violates strict aliasing rule
```

### Exceptions (Allowed Aliasing)

You CAN access any object through:
- `char*`, `unsigned char*`, `std::byte*`
- A pointer to the object's actual type
- A pointer to a base class type

```cpp
int x = 42;
unsigned char* bytes = reinterpret_cast<unsigned char*>(&x);

// OK! Can examine bytes
for (size_t i = 0; i < sizeof(int); ++i) {
    std::cout << static_cast<int>(bytes[i]) << " ";
}
```

## Safe Alternative: std::bit_cast (C++20)

```cpp
#include <bit>

float f = 3.14f;
uint32_t bits = std::bit_cast<uint32_t>(f);  // Safe type punning

// Or use memcpy (pre-C++20)
uint32_t bits2;
std::memcpy(&bits2, &f, sizeof(float));
```

## What reinterpret_cast CANNOT Do

```cpp
// Cannot cast away const (use const_cast)
const int* cp = new int(42);
// int* p = reinterpret_cast<int*>(cp);  // Error!

// Cannot convert between function and data pointers (usually)
void (*fp)() = myFunction;
// int* ip = reinterpret_cast<int*>(fp);  // Implementation-defined
```

## Comparison with Other Casts

| Cast | When to Use |
|------|-------------|
| `static_cast` | Related type conversions (numeric, hierarchy) |
| `dynamic_cast` | Safe downcasting with runtime check |
| `const_cast` | Add/remove const |
| `reinterpret_cast` | Bit-level reinterpretation (last resort) |

## Red Flags

If you're using `reinterpret_cast`, ask yourself:
1. Is there a safer alternative?
2. Am I violating strict aliasing?
3. Is this portable across platforms?
4. Have I documented why this is necessary?

## Key Takeaways

- Most dangerous cast—no type checking whatsoever
- Used for low-level pointer/integer conversions
- Violating strict aliasing is undefined behavior
- Safe to alias through `char*` or `unsigned char*`
- Use `std::bit_cast` (C++20) for safe type punning
- Always document and justify usage

## Common Interview Questions

> [!question]- What is the strict aliasing rule?
> You cannot access an object through a pointer of a different type (with exceptions for char types). Compilers assume pointers of different types don't alias, enabling optimizations.

> [!question]- When would you use reinterpret_cast?
> Low-level systems programming: memory-mapped I/O, serialization, debugger tools, FFI with C libraries, examining raw memory bytes.

> [!question]- What's wrong with this code?
> ```cpp
> int x = 0x12345678;
> short* sp = reinterpret_cast<short*>(&x);
> short s = *sp;  // ?
> ```
> This violates strict aliasing (UB). Also, the result depends on endianness—not portable.

## Related Topics

- [[01_static_cast|static_cast]]
- [[05_c_style_vs_cpp_casts|C-Style vs C++ Casts]]
