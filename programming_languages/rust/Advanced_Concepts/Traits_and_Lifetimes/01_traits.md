# Traits

Traits are Rust's way of defining **shared behavior**. They are similar to interfaces in Java or abstract base classes in C++.

A trait defines a set of methods that a type must implement.

---

## Defining and Implementing a Trait

```rust
// Define a trait
pub trait Summary {
    fn summarize(&self) -> String;
}

// A type
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// Implement the trait for the type
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

### Default Implementations

Traits can provide default method implementations.

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    // Default implementation that calls another method
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

---

## Traits as Parameters (Trait Bounds)

This is how Rust achieves polymorphism. You write functions that accept "any type that implements a certain trait."

### `impl Trait` Syntax (Simpler)

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

### Trait Bound Syntax (More Flexible)

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// Multiple bounds
pub fn notify<T: Summary + Display>(item: &T) { ... }

// `where` clause for readability
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

### Comparison to Other Languages

| Rust Trait Bounds | Java Generics | C++ Templates |
| :---------------- | :------------ | :------------ |
| `fn foo<T: Clone>(t: T)` | `<T extends Cloneable> void foo(T t)` | `template<typename T> void foo(T t)` (uses concepts in C++20) |
| Compile-time checked | Compile-time checked (with type erasure at runtime) | Compile-time checked |
| Zero-cost abstraction (monomorphization) | Some runtime overhead | Zero-cost abstraction |

---

## Common Standard Library Traits

| Trait | Purpose | Example Use |
| :---- | :------ | :---------- |
| `Clone` | Explicit deep copy | `let y = x.clone();` |
| `Copy` | Implicit bitwise copy (for simple types) | `let y = x;` (x still valid) |
| `Debug` | Formatting for debugging (`:?`) | `println!("{:?}", my_struct);` |
| `Display` | User-facing formatting (`{}`) | `println!("{}", my_struct);` |
| `Default` | Create a default value | `let opts = Config::default();` |
| `PartialEq`, `Eq` | Equality comparison (`==`) | `if a == b { ... }` |
| `PartialOrd`, `Ord` | Ordering comparison (`<`, `>`) | `vec.sort();` |
| `Iterator` | Iteration protocol | `for item in collection { ... }` |

### Deriving Traits

Many common traits can be automatically implemented by the compiler using `#[derive(...)]`.

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```
