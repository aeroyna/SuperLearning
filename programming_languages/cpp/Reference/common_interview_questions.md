# Common C++ Interview Questions

Quick answers to frequently asked C++ interview questions.

## Language Basics

### What's the difference between struct and class?
Default access: struct is public, class is private. That's the only technical difference.

### What are the four types of casts in C++?
- `static_cast`: compile-time, related types
- `dynamic_cast`: runtime, safe downcasting (needs polymorphic type)
- `const_cast`: add/remove const
- `reinterpret_cast`: bit-level reinterpretation

### What is RAII?
Resource Acquisition Is Initialization. Resources are acquired in constructors and released in destructors. Ensures cleanup even if exceptions occur.

### What's the difference between `new`/`delete` and `malloc`/`free`?
`new`/`delete` call constructors/destructors, are type-safe, and can be overloaded. `malloc`/`free` just allocate raw memory.

## Object-Oriented Programming

### What is a virtual function?
A function that can be overridden in derived classes and uses dynamic dispatch (vtable lookup) when called through base pointer/reference.

### Why make destructors virtual?
To ensure derived class destructors are called when deleting through a base pointer, preventing resource leaks.

### What is the diamond problem?
When a class inherits from two classes that share a common base, causing ambiguity and duplicate base instances. Solved with virtual inheritance.

### What is object slicing?
When copying a derived object to a base object by value, losing the derived part. Prevent by using pointers/references.

### What's the Rule of Three/Five/Zero?
- **Three**: If you define destructor, copy ctor, or copy assignment, define all three
- **Five**: Add move ctor and move assignment
- **Zero**: Best - don't define any, use RAII types

## Memory Management

### What are smart pointers?
- `unique_ptr`: Exclusive ownership, no copies
- `shared_ptr`: Shared ownership via reference counting
- `weak_ptr`: Non-owning reference to shared_ptr's object

### What's the difference between stack and heap?
Stack: automatic, fast, limited size, LIFO. Heap: manual (or smart ptr), slower, larger, any order.

### What is a memory leak?
Allocated memory that's never freed. Prevented by RAII and smart pointers.

### What is a dangling pointer?
A pointer to memory that has been freed. Accessing it is undefined behavior.

## Modern C++ (C++11 and later)

### What are rvalues and lvalues?
- **Lvalue**: Has identity, can take address (`&x` works)
- **Rvalue**: Temporary, no persistent address (like `x + y`)

### What is move semantics?
Allows "stealing" resources from temporary objects instead of copying. Uses rvalue references (`&&`) and move constructors.

### What is `auto`?
Automatic type deduction from initializer.

### What are lambda expressions?
Anonymous function objects: `[captures](params) { body }`

### What is `nullptr` vs `NULL`?
`nullptr` is type-safe (has type `std::nullptr_t`). `NULL` is just `0` and can cause ambiguity.

## Templates

### What is template instantiation?
When the compiler generates actual code from a template for specific types.

### What is SFINAE?
Substitution Failure Is Not An Error. If template substitution fails, that overload is removed from consideration rather than causing an error.

### What's the difference between `class` and `typename` in templates?
Mostly interchangeable. Use `typename` to indicate dependent types.

## Concurrency

### What is a race condition?
When program behavior depends on timing of threads accessing shared data. Fixed with synchronization (mutexes).

### What is a deadlock?
When threads wait for each other in a cycle, each holding resources the other needs.

### What's the difference between `mutex` and `atomic`?
Mutex: locks for critical sections. Atomic: lock-free operations on single variables.

## STL

### vector vs array?
`vector`: dynamic size, heap. `array`: fixed size, stack/inline.

### map vs unordered_map?
`map`: sorted, O(log n), uses tree. `unordered_map`: unsorted, O(1) average, uses hash.

### How does `std::sort` work?
Typically introsort: quicksort + heapsort + insertion sort. O(n log n) guaranteed.

## Quick Facts

| Topic | Key Point |
|-------|-----------|
| const member functions | Can be called on const objects |
| static members | Belong to class, not instances |
| friend | Grants private access |
| virtual | Enables polymorphism |
| override | Documents intent, catches errors |
| final | Prevents further override/inheritance |
| explicit | Prevents implicit conversions |
| constexpr | Compile-time evaluation |
| noexcept | Promises no exceptions |
| inline | Hint for inlining |
