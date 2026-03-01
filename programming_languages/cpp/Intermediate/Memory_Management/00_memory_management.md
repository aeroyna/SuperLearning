# Memory Management

Memory management is the cornerstone of high-performance C++ programming. Unlike managed languages with garbage collectors, C++ gives you direct control over memory allocation and deallocation. This power comes with the responsibility to avoid leaks and corruption. This chapter covers the fundamental differences between stack and heap memory, introduces modern tools like smart pointers to automate memory management, and explains the RAII idiom, which is central to writing safe and idiomatic C++ code.

You will learn about:
- **Stack vs. Heap:** Understanding memory regions, lifetime, and allocation costs.
- **Smart Pointers:** Using `std::unique_ptr`, `std::shared_ptr`, and `std::weak_ptr` to manage ownership automatically.
- **RAII:** Resource Acquisition Is Initialization—binding resource lifecycle to object lifetime.

## In this chapter

- **[Stack and Heap](Stack_and_Heap/00_stack_and_heap.md)**
- **[Smart Pointers](Smart_Pointers/00_smart_pointers.md)**
- **[RAII](RAII/00_raii.md)**
- **[Practice Problems](practice_problems.md)**
