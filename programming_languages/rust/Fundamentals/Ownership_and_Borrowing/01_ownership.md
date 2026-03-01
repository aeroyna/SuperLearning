# Ownership and Borrowing

This is Rust's most unique feature and the source of its memory safety guarantees **without a garbage collector**. Understanding Ownership deeply is *essential* before progressing further.

## Why Does This Exist?

Consider how other languages handle memory:

| Language | Memory Management | Trade-off |
| :------- | :---------------- | :-------- |
| **C/C++** | Manual (`malloc`/`free`, `new`/`delete`) | Fast, but prone to dangling pointers, double frees, and memory leaks. |
| **Java/Python/Go** | Garbage Collection (GC) | Safe, but introduces runtime overhead and non-deterministic pauses. |
| **Rust** | **Ownership** | Safe *and* no GC. Memory is freed predictably when ownership ends. |

Rust's Ownership system is a set of compile-time rules that the borrow checker enforces. If your code violates these rules, **it will not compile**.

---

## The 3 Rules of Ownership

Memorize these:

1.  Each value in Rust has a variable that's called its **owner**.
2.  There can only be **one owner** at a time.
3.  When the owner goes out of **scope**, the value will be **dropped** (its memory is freed).

### Example: Stack vs. Heap

Understanding *where* data lives is critical.

```rust
fn main() {
    let x = 5;          // x is stored on the STACK (fixed size, copied)
    let s = String::from("hello"); // s is stored on the HEAP (growable)
} // Both x and s go out of scope. The String's heap memory is DROPPED (freed).
```

**Memory Diagram:**

```
Stack:                 Heap:
+-------+----------+   +---+---+---+---+---+
| name  |   value  |   | h | e | l | l | o |
+-------+----------+   +---+---+---+---+---+
|   x   |     5    |    ^
+-------+----------+    |
|   s   | ptr ------+---+
|       | len: 5   |
|       | cap: 5   |
+-------+----------+
```
The `String` struct on the stack holds a pointer, length, and capacity. The actual character data is on the heap.

---

## Move Semantics

When you assign a heap-allocated value to a new variable, Rust "**moves**" it. The original variable becomes **invalid**.

```rust
let s1 = String::from("hello");
let s2 = s1; // Ownership is MOVED from s1 to s2.

println!("{}", s1); // COMPILE ERROR! "value borrowed here after move"
```

**Why?** This prevents a **double free**. If both `s1` and `s2` were valid, the heap memory would be freed twice when they both go out of scope, which is Undefined Behavior.

### Comparison to Other Languages

| Language | What happens with `s2 = s1`? |
| :------- | :--------------------------- |
| **C++** | Deep copy by default (slow), or use `std::move` for similar behavior. |
| **Java** | `s2` is a new *reference* to the *same object*. The GC manages the lifetime. |
| **Python** | `s2` is a new *reference* to the *same object*. Reference counting + GC. |
| **Rust** | `s1` is **invalidated**. Only `s2` owns the data now. |

### The `Copy` Trait

Simple types on the stack (integers, booleans, floats, chars) implement the `Copy` trait. They are cheap to copy, so they don't move; they are duplicated.

```rust
let x = 5;
let y = x; // y is a COPY of x, not a move.
println!("x = {}, y = {}", x, y); // Both are valid!
```

---

## Borrowing (References)

Moving is often inconvenient. You frequently want to *access* data without taking ownership. You do this by creating a **reference** (`&`), which is called **borrowing**.

A reference is like a pointer, but the compiler guarantees it is always valid.

### The Rules of Borrowing

These rules prevent data races at compile time:

1.  At any given time, you can have **either** (but not both):
    *   **One mutable reference** (`&mut T`).
    *   **Any number of immutable references** (`&T`).
2.  References must always be **valid** (no dangling pointers).

### Immutable Borrowing (`&T`)

Allows reading data. You can have many immutable references at once.

```rust
fn calculate_length(s: &String) -> usize { // s is a REFERENCE to a String
    s.len()
} // s goes out of scope, but since it's just a reference, nothing is dropped.

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // Pass a reference, don't move s1
    println!("The length of '{}' is {}.", s1, len); // s1 is still valid!
}
```

### Mutable Borrowing (`&mut T`)

Allows modifying data. You can only have **one** mutable reference at a time. This prevents data races.

```rust
fn change(s: &mut String) {
    s.push_str(", world");
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s); // Prints "hello, world"
}
```

### Why the Restriction?

This rule prevents **data races**. A data race occurs when:
1.  Two or more pointers access the same data at the same time.
2.  At least one of them is writing.
3.  There's no synchronization.

Rust's borrow checker prevents this at **compile time**.

```rust
let mut s = String::from("hello");

let r1 = &s;     // OK - immutable borrow
let r2 = &s;     // OK - multiple immutable borrows are fine
// let r3 = &mut s; // COMPILE ERROR! Cannot borrow `s` as mutable while immutable refs exist.

println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point (their scope ends).

let r3 = &mut s; // OK - now we can borrow as mutable
println!("{}", r3);
```

---

## Common Pitfalls for Newcomers

### 1. Trying to use a moved value
**Solution:** Clone the data if you need two owners (`.clone()`), or use references.

### 2. Returning a reference to a local variable
This creates a **dangling pointer**.
```rust
// fn dangle() -> &String {
//     let s = String::from("hello");
//     &s // ERROR: `s` is dropped here, the reference would be invalid.
// }

// Solution: Return the owned value, transferring ownership OUT.
fn no_dangle() -> String {
    let s = String::from("hello");
    s // Ownership is moved out of the function
}
```

### 3. Mutating while iterating
```rust
let mut v = vec![1, 2, 3];
// for i in &v {
//     v.push(*i * 2); // ERROR: cannot borrow `v` as mutable while also borrowed as immutable
// }
```
**Solution:** Collect changes first, then apply, or use iterators like `iter_mut()`.
