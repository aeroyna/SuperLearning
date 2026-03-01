# Smart Pointers

Smart pointers are data structures that act like pointers but also have additional metadata and capabilities. In Rust, they are types that implement the `Deref` and `Drop` traits.

---

## `Box<T>`: Heap Allocation

`Box<T>` is the simplest smart pointer. It allocates data on the **heap** instead of the stack.

### Why Use `Box`?

1.  **Data of unknown size at compile time:** When you have a type whose size can't be known at compile time (like a recursive type).
2.  **Transferring ownership of large data:** Moving a `Box` only copies the pointer (cheap), not the heap data.
3.  **Trait objects:** To own a value that implements a specific trait.

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b); // b = 5
} // `b` is dropped here, and its heap data is freed.
```

### Recursive Types

The classic use case. A linked list needs `Box` because the size of a `List` would otherwise be infinite.

```rust
enum List {
    Cons(i32, Box<List>), // Box breaks the infinite recursion
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

---

## `Rc<T>`: Reference Counting

`Rc<T>` (Reference Counted) enables **multiple ownership**. It keeps track of how many owners exist, and the data is only cleaned up when the count reaches zero.

> **Important:** `Rc` is for **single-threaded** scenarios only. It has no thread-safety overhead.

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(String::from("hello"));
    let b = Rc::clone(&a); // Increments ref count, doesn't deep clone data
    let c = Rc::clone(&a);

    println!("Reference count: {}", Rc::strong_count(&a)); // Prints 3
}
```

### Memory Diagram

```
Stack:              Heap:
+-----+             +-------------------+
| a  ---+---------->| ref_count: 3      |
+-----+ |           | data: "hello"     |
| b  ---+           +-------------------+
+-----+ |
| c  ---+
+-----+
```

---

## `Arc<T>`: Atomic Reference Counting

`Arc<T>` (Atomically Reference Counted) is the **thread-safe** version of `Rc`. It uses atomic operations to update the reference count, which has a small performance cost.

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let s = Arc::new(String::from("shared data"));
    let mut handles = vec![];

    for _ in 0..10 {
        let s_clone = Arc::clone(&s);
        let handle = thread::spawn(move || {
            println!("{}", s_clone);
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }
}
```

---

## `RefCell<T>`: Interior Mutability

`RefCell<T>` allows you to mutate data even when there are immutable references to it. It enforces Rust's borrowing rules **at runtime** instead of compile time.

If you violate the rules (e.g., two mutable borrows), the program **panics** instead of failing to compile.

```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(5);

    // Get a mutable reference (checked at runtime)
    *data.borrow_mut() += 1;

    println!("{}", data.borrow()); // Prints 6
}
```

### `Rc<RefCell<T>>`: The Pattern

Combining `Rc` (multiple owners) with `RefCell` (mutability) gives you a value that can have multiple owners *and* be mutated.

```rust
use std::cell::RefCell;
use std::rc::Rc;

let value = Rc::new(RefCell::new(5));

let a = Rc::clone(&value);
let b = Rc::clone(&value);

*a.borrow_mut() += 10;
*b.borrow_mut() += 20;

println!("{}", value.borrow()); // Prints 35
```

---

## Summary Table

| Pointer | Ownership | Thread-Safe? | Mutability Check |
| :------ | :-------- | :----------- | :--------------- |
| `Box<T>` | Single | N/A (no sharing) | Compile-time |
| `Rc<T>` | Multiple | No | Compile-time (immutable) |
| `Arc<T>` | Multiple | Yes | Compile-time (immutable) |
| `RefCell<T>` | Single | No | **Runtime** |
| `Mutex<T>` | (see Concurrency) | Yes | **Runtime** |
