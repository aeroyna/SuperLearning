# Structs and Enums

Rust provides powerful ways to define custom data types. Unlike classes in OOP languages, Rust separates **data** (structs/enums) from **behavior** (impl blocks).

---

## Structs

Structs are used to create custom data types that group related fields.

### Defining and Instantiating a Struct

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someuser"),
        active: true,
        sign_in_count: 1,
    };
}
```

### Memory Layout

Structs are laid out contiguously in memory (like C structs), which is great for cache performance.

```
Stack (assuming owned String fields point to heap):
+-----------------+---------------------------+
| Field           | Value                     |
+-----------------+---------------------------+
| username (ptr)  | -> Heap: "someuser"       |
| username (len)  | 8                         |
| username (cap)  | 8                         |
| email (ptr)     | -> Heap: "some..."        |
| email (len)     | 19                        |
| email (cap)     | 19                        |
| sign_in_count   | 1                         |
| active          | true (1 byte, padded)     |
+-----------------+---------------------------+
```

### Methods (`impl` blocks)

You add behavior to a struct using `impl` blocks. The first parameter `&self` is a reference to the instance (like `this` in Java/C++).

```rust
impl User {
    // Associated function (like a static method or constructor)
    fn new(email: String, username: String) -> Self {
        Self {
            email,
            username,
            active: true,
            sign_in_count: 0,
        }
    }

    // Method (takes &self)
    fn is_active(&self) -> bool {
        self.active
    }
}

let user = User::new(String::from("a@b.com"), String::from("user1"));
println!("Active: {}", user.is_active());
```

### Comparison to Other Languages

| Rust Struct | C++ `struct`/`class` | Java `class` | Python `class` |
| :---------- | :------------------- | :----------- | :------------- |
| Data only, `impl` for methods | Data + methods in one | Data + methods in one | Data + methods in one |
| No inheritance | Has inheritance | Has inheritance | Has inheritance |
| Uses Traits for polymorphism | Uses virtual methods | Uses interfaces/abstract classes | Duck typing |

---

## Enums

Rust Enums are **Algebraic Data Types (ADTs)**. Each variant can hold different types and amounts of data. This is far more powerful than C/Java enums (which are just named integers).

### Basic Enum

```rust
enum IpAddrKind {
    V4,
    V6,
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

### Enums with Data

Each variant can hold different data structures.

```rust
enum Message {
    Quit,                       // No data
    Move { x: i32, y: i32 },    // Named fields (like a struct)
    Write(String),              // A single String
    ChangeColor(i32, i32, i32), // Three i32 values
}
```

### Memory Layout (Tagged Union)

An enum is represented as a **tagged union** (or discriminated union). The size of an enum is the size of its **largest variant** plus a **tag** (discriminant) to identify which variant is active.

```
Enum in memory (conceptual):
+-------+-----------------------------------+
| Tag   | Payload (sized to largest variant)|
+-------+-----------------------------------+
| 0     | (empty for Quit)                  |
| 1     | x: i32, y: i32 (for Move)         |
| 2     | String ptr, len, cap (for Write)  |
| 3     | i32, i32, i32 (for ChangeColor)   |
+-------+-----------------------------------+
```

---

## Pattern Matching (`match`)

Enums are designed to be used with `match`, Rust's powerful control flow construct. `match` is **exhaustive**—you must handle *every* variant.

```rust
fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("Quit"),
        Message::Move { x, y } => println!("Move to ({}, {})", x, y),
        Message::Write(text) => println!("Text: {}", text),
        Message::ChangeColor(r, g, b) => println!("Color: {}, {}, {}", r, g, b),
    }
}
```

---

## The Critical Enums: `Option<T>` and `Result<T, E>`

Rust's standard library uses two enums so frequently that they are part of the prelude.

### `Option<T>`: Replacing `null`

Rust has **no null**. Instead, the absence of a value is explicitly represented by `Option`.

```rust
enum Option<T> {
    Some(T),
    None,
}

fn divide(numerator: f64, denominator: f64) -> Option<f64> {
    if denominator == 0.0 {
        None // Explicitly stating there's no result
    } else {
        Some(numerator / denominator)
    }
}
```

**Why is this better than `null`?**
The compiler **forces** you to handle the `None` case. You cannot accidentally use a `None` value as if it were `Some`.

| Rust `Option<T>` | Java `Optional<T>` | C++ `std::optional<T>` | C#/JavaScript `null` |
| :--------------- | :----------------- | :--------------------- | :------------------- |
| Compile-time check | Runtime check (`.get()` can throw) | Runtime check (`.value()` throws) | Runtime `NullPointerException` / `TypeError` |

### `Result<T, E>`: Error Handling

Used for operations that can fail. `T` is the success type, `E` is the error type.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

use std::fs::File;

fn main() {
    let f = File::open("hello.txt"); // Returns Result<File, io::Error>

    let file = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening file: {:?}", error),
    };
}
```

### The `?` Operator

A shortcut for propagating errors. If the `Result` is `Err`, it returns early from the function.

```rust
use std::io::{self, Read};
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?; // Returns Err if file not found
    let mut s = String::new();
    f.read_to_string(&mut s)?; // Returns Err if read fails
    Ok(s)
}
```
