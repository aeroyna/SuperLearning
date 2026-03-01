# Creating Threads

There are multiple ways to create threads in Java. While the syntax is simple, understanding the *implications* of each approach regarding code coupling and resource management is key.

## 1. Extending `Thread` Class
The most basic, direct way.

```java
class Worker extends Thread {
    @Override
    public void run() {
        System.out.println("Work running in: " + Thread.currentThread().getName());
    }
}

// Usage
Worker w = new Worker();
w.start(); // NEVER call run() directly!
```

### Why avoid this?
1.  **Single Inheritance Limit:** Java only allows extending one class. If you extend `Thread`, you can't extend anything else (like `BaseService`).
2.  **Coupling:** It tightly couples your "Task" (the logic in `run`) with the "Runner" (the thread mechanism). Good design separates the "What" (Runnable) from the "How" (Thread).

## 2. Implementing `Runnable` Interface
The standard, preferred way for raw threads.

```java
class Task implements Runnable {
    @Override
    public void run() {
        System.out.println("Task running");
    }
}

// Usage
Thread t = new Thread(new Task()); // Pass task to thread
t.start();
```

### Advantages
1.  **Decoupling:** The `Task` is just a job. It can be run by a `Thread`, a `ThreadPool`, or even synchronously on the main thread.
2.  **Inheritance:** Your `Task` class is free to extend another class.
3.  **Resource Sharing:** Multiple `Thread` objects can be initialized with the *same* `Runnable` instance, sharing the instance variables of that Runnable (though be careful of thread safety!).

## 3. Implementing `Callable` (Java 5+)
Both `Thread` and `Runnable` suffer from two flaws:
1.  They cannot return a result.
2.  They cannot throw Checked Exceptions (you must try-catch inside `run`).

`Callable<V>` solves this. It is generally used with the Executor Framework (covered later), not raw threads.

```java
Callable<String> task = () -> {
    if (someCondition) throw new IOException("Failed");
    return "Result";
};
```

## 4. Pitfall: `run()` vs `start()`

```java
Thread t = new Thread(new Task());
t.run();   // <--- WRONG!
t.start(); // <--- RIGHT
```

*   **`t.run()`:** This is just a normal method call. The code executes on the **CURRENT** thread (e.g., main thread). No new thread is spawned. This is a common bug.
*   **`t.start()`:** This makes a system call to the OS to allocate a new thread context. The OS then calls the `run()` method inside that new context.

## 5. Thread Names
**Always** name your threads. When you look at logs or thread dumps, `Thread-0`, `Thread-1` is useless. `Order-Processor-Thread-1` tells you exactly what crashed.

```java
Thread t = new Thread(new Task(), "Order-Processor");
```