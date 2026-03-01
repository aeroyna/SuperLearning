# Introduction to Smart Pointers

Smart pointers are objects that act like pointers but provide automatic memory management. They are part of the C++ Standard Library (since C++11) and are defined in the `<memory>` header.

## Why use Smart Pointers?

Manual memory management using raw pointers (`new` and `delete`) is a common source of bugs in C++:

*   **Memory Leaks:** Forgetting to call `delete` on a dynamically allocated object.
*   **Dangling Pointers:** Accessing memory that has already been deallocated.
*   **Double Deletes:** Calling `delete` twice on the same pointer.

Smart pointers help to solve these problems by automatically managing the lifetime of the object they point to. They follow the **RAII (Resource Acquisition Is Initialization)** principle, which we will discuss in more detail later.

## Types of Smart Pointers

There are three main types of smart pointers in C++:

1.  **`std::unique_ptr`:** Represents unique ownership of a resource. Only one `unique_ptr` can point to a given object at a time. When the `unique_ptr` is destroyed, the object it points to is also destroyed.

2.  **`std::shared_ptr`:** Represents shared ownership of a resource. Multiple `shared_ptr`s can point to the same object. The object is destroyed only when the last `shared_ptr` pointing to it is destroyed. This is managed through a reference count.

3.  **`std::weak_ptr`:** A non-owning smart pointer that holds a "weak" reference to an object managed by a `std::shared_ptr`. It is used to break circular dependencies between `std::shared_ptr`s.

In the next sections, we will look at each of these smart pointers in detail. As a general rule, you should prefer `std::unique_ptr` by default, and use `std::shared_ptr` only when you need to share ownership.
