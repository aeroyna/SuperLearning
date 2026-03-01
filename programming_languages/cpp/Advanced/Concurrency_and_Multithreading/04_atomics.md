# Atomics

An atomic operation is an operation that is guaranteed to be executed without being interrupted by any other thread. The C++ standard library provides tools for atomic operations in the `<atomic>` header.

## Why use Atomics?

Consider the simple operation `counter++`. This is not an atomic operation. It typically involves three steps:
1.  Read the value of `counter` from memory into a register.
2.  Increment the value in the register.
3.  Write the new value back to memory.

Another thread could interrupt this sequence at any point, leading to a race condition. You could use a mutex to protect the `counter++` operation, but for simple types like integers, atomics can be a more efficient, "lock-free" alternative.

## `std::atomic`

`std::atomic` is a class template that can be used to make any type atomic.

```cpp
#include <atomic>

std::atomic<int> counter;
```

### Example

Let's rewrite the counter example from the mutex section using `std::atomic`.

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <atomic>

std::atomic<int> shared_counter(0);

void increment() {
    for (int i = 0; i < 10000; ++i) {
        shared_counter++; // this is now an atomic operation
    }
}

int main() {
    std::vector<std::thread> threads;
    for (int i = 0; i < 10; ++i) {
        threads.push_back(std::thread(increment));
    }

    for (auto& th : threads) {
        th.join();
    }

    std::cout << "Final counter value: " << shared_counter << std::endl;
    return 0;
}
```
This code is now thread-safe without using a mutex.

## Memory Ordering

Atomics are not just about indivisible operations; they are also about memory ordering. Memory ordering specifies how memory operations in one thread are seen by other threads.

This is a very complex topic, but the default memory ordering for all atomic operations is **sequentially consistent** (`std::memory_order_seq_cst`). This is the strongest and most intuitive memory order, and it guarantees that all threads will see the operations in the same order.

For performance-critical applications, you can use more relaxed memory orders (e.g., `std::memory_order_relaxed`, `std::memory_order_acquire`, `std::memory_order_release`), but this is an advanced topic that requires a deep understanding of how memory works on modern CPUs. For most use cases, the default sequential consistency is sufficient.

## `std::atomic_flag`

`std::atomic_flag` is a simple boolean atomic type. It is guaranteed to be lock-free, and it can be used to implement simple spinlocks.

## When to use Atomics

Use atomics for simple operations on primitive types (like counters or flags) where the performance overhead of a mutex is not acceptable. For more complex operations or for protecting larger data structures, a mutex is still the appropriate tool.
