# Practice Problems: Concurrency and Multithreading

## Problem 1: Parallel Sum

Write a function that calculates the sum of the elements of a `std::vector<int>` in parallel using multiple threads. The function should take the vector and the number of threads to use as input.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <numeric>
#include <atomic>

// This function will be executed by each thread
void partial_sum(const std::vector<int>& vec, size_t start, size_t end, std::atomic<long long>& result) {
    long long local_sum = 0;
    for (size_t i = start; i < end; ++i) {
        local_sum += vec[i];
    }
    result += local_sum;
}

long long parallel_sum(const std::vector<int>& vec, unsigned int num_threads) {
    std::atomic<long long> total_sum(0);
    std::vector<std::thread> threads;
    size_t block_size = (vec.size() + num_threads - 1) / num_threads;

    for (unsigned int i = 0; i < num_threads; ++i) {
        size_t start = i * block_size;
        size_t end = std::min(start + block_size, vec.size());
        if (start < end) {
            threads.emplace_back(partial_sum, std::ref(vec), start, end, std::ref(total_sum));
        }
    }

    for (auto& th : threads) {
        th.join();
    }

    return total_sum;
}

int main() {
    std::vector<int> numbers(1000000, 1); // a large vector of 1s
    unsigned int num_threads = std::thread::hardware_concurrency();

    long long sum = parallel_sum(numbers, num_threads);

    std::cout << "Sum: " << sum << std::endl;
    std::cout << "Expected sum: " << numbers.size() << std::endl;

    return 0;
}
```
*Note: Using `std::atomic` for the final sum is a simple way to do this. For even better performance on a real-world problem, you might have each thread return its partial sum (e.g., via a `std::future`) and then sum the partial sums in the main thread.*

## Problem 2: Thread-safe Queue

Implement a simple thread-safe queue class. It should have `push` and `pop` methods that can be called by multiple threads concurrently.

### Solution

```cpp
#include <iostream>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <thread>

template <typename T>
class ThreadSafeQueue {
private:
    std::queue<T> queue;
    std::mutex mtx;
    std::condition_variable cv;

public:
    void push(T value) {
        std::lock_guard<std::mutex> lock(mtx);
        queue.push(value);
        cv.notify_one();
    }

    T pop() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this] { return !queue.empty(); });
        T value = queue.front();
        queue.pop();
        return value;
    }
};

void producer(ThreadSafeQueue<int>& q) {
    for (int i = 0; i < 10; ++i) {
        q.push(i);
        std::cout << "Pushed " << i << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void consumer(ThreadSafeQueue<int>& q) {
    for (int i = 0; i < 10; ++i) {
        int val = q.pop();
        std::cout << "Popped " << val << std::endl;
    }
}

int main() {
    ThreadSafeQueue<int> q;
    std::thread p(producer, std::ref(q));
    std::thread c(consumer, std::ref(q));

    p.join();
    c.join();

    return 0;
}
```
This problem demonstrates the use of mutexes and condition variables to create a thread-safe data structure.
