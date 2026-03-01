# Thread Libraries\n\n## POSIX Pthreads

Standard API for thread creation and synchronization.

```c
#include <pthread.h>

pthread_t thread;
pthread_create(&thread, NULL, function, arg);
pthread_join(thread, NULL);
pthread_mutex_lock(&mutex);
pthread_mutex_unlock(&mutex);
```

## Windows Threads

```cpp
#include <windows.h>

HANDLE thread = CreateThread(NULL, 0, ThreadFunc, NULL, 0, NULL);
WaitForSingleObject(thread, INFINITE);
CloseHandle(thread);
```

## Java Threads

```java
// Method 1: Extend Thread
class MyThread extends Thread {
    public void run() { /* work */ }
}
new MyThread().start();

// Method 2: Implement Runnable
new Thread(() -> { /* work */ }).start();
```

## Python Threading

```python
import threading

t = threading.Thread(target=function, args=(arg,))
t.start()
t.join()
```

## Key Takeaways

1. Pthreads: POSIX standard, portable across Unix/Linux
2. Windows: Native threading API
3. Java: Platform-independent, high-level
4. Python: GIL limits true parallelism

## Interview Focus

1. Compare different thread libraries
2. How to create thread in C/Java/Python?
3. Synchronization primitives in each library?
4. What is Python GIL and its impact?
