# Concurrency

Rust's approach to concurrency is often called **"Fearless Concurrency"**. The ownership and type system ensure that many concurrency bugs (like data races) are caught at **compile time**.

---

## Creating Threads

Use `std::thread::spawn` to create a new thread. It takes a closure that contains the code to run.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap(); // Wait for the spawned thread to finish
}
```

### `move` Closures

If the spawned thread needs to use data from the main thread, you must use `move` to transfer ownership into the closure.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    // println!("{:?}", v); // COMPILE ERROR: v was moved

    handle.join().unwrap();
}
```

---

## Message Passing: Channels (MPSC)

Rust's standard library provides **channels** for sending messages between threads. Think of it as a one-way pipe.

`mpsc` stands for **M**ultiple **P**roducers, **S**ingle **C**onsumer.

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel(); // tx = transmitter, rx = receiver

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        // println!("{}", val); // COMPILE ERROR: val was moved into the channel
    });

    let received = rx.recv().unwrap(); // Blocks until a message is received
    println!("Got: {}", received);
}
```

### Sending Multiple Values

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec!["hi", "from", "the", "thread"];
        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    // Treat rx as an iterator
    for received in rx {
        println!("Got: {}", received);
    }
}
```

---

## Shared-State Concurrency: `Mutex<T>`

A **Mutex** (Mutual Exclusion) allows only one thread to access data at a time.

### `Mutex<T>` API

1.  To access the data, a thread must first acquire the **lock** by calling `.lock()`.
2.  `.lock()` returns a `MutexGuard`, which provides access to the data.
3.  When the `MutexGuard` goes out of scope, the lock is **automatically released**.

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap(); // Acquire lock
        *num = 6;
    } // Lock is released here (MutexGuard is dropped)

    println!("m = {:?}", m); // Prints m = Mutex { data: 6 }
}
```

### Sharing a `Mutex` Between Threads: `Arc<Mutex<T>>`

To share a `Mutex` across multiple threads, wrap it in an `Arc` (Atomically Reference Counted).

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap()); // Prints 10
}
```

---

## `Send` and `Sync` Traits

These marker traits are the foundation of Rust's concurrency safety.

| Trait | Meaning |
| :---- | :------ |
| `Send` | A type is safe to **transfer ownership** to another thread. |
| `Sync` | A type is safe to be **referenced** from multiple threads. (i.e., `&T` is `Send`) |

Almost all primitive types are `Send` and `Sync`. Types like `Rc<T>` are *not* `Send` because they don't have thread-safe reference counting.
