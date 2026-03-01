# Smart Pointers

Smart pointers are wrapper classes that manage pointers to ensure that memory is automatically released when it is no longer needed. Introduced in C++11, they are the preferred way to handle dynamic memory, replacing raw `new` and `delete` in most scenarios. This chapter details the three main types of smart pointers, their ownership models, and how they prevent common issues like memory leaks and dangling pointers.

You will learn about:
- **`std::unique_ptr`:** Exclusive ownership and zero-overhead abstraction.
- **`std::shared_ptr`:** Shared ownership and reference counting.
- **`std::weak_ptr`:** Non-owning references to break cycles.

## In this chapter

- **[Smart Pointers Intro](01_smart_pointers_intro.md)**
- **[Unique Ptr](02_unique_ptr.md)**
- **[Shared Ptr](03_shared_ptr.md)**
- **[Weak Ptr](04_weak_ptr.md)**
