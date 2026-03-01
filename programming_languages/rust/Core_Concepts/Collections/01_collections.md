# Collections

Rust's standard library provides a set of common data structures called collections. Unlike stack-allocated types, collections store their data on the **heap**, meaning they can grow and shrink at runtime.

The three most common collections are `Vec<T>`, `String`, and `HashMap<K, V>`.

---

## Vectors (`Vec<T>`)

A vector is a growable, contiguous list of elements. It's Rust's equivalent to `std::vector` in C++, `ArrayList` in Java, or a Python `list`.

### Creating and Modifying Vectors

```rust
// Create an empty vector
let mut v: Vec<i32> = Vec::new();

// Push elements
v.push(1);
v.push(2);
v.push(3);

// Create with the vec! macro
let v2 = vec![1, 2, 3];
```

### Accessing Elements

```rust
let v = vec![1, 2, 3, 4, 5];

// Using indexing (PANICS if out of bounds)
let third: &i32 = &v[2];
println!("The third element is {}", third);

// Using .get() (Returns Option<&T>, SAFE)
match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
```

### Memory Layout and Growth Strategy

A `Vec` is three components on the **stack**: a pointer to heap memory, a length, and a capacity.

```
Stack:          Heap:
+-------+       +---+---+---+---+---+---+---+---+
| ptr --------->| 1 | 2 | 3 | 4 | 5 |   |   |   |
| len: 5 |      +---+---+---+---+---+---+---+---+
| cap: 8 |      ^ length = 5        ^ capacity = 8
+-------+
```

When you `push` and the length exceeds capacity, the vector **reallocates**:
1.  A new, larger buffer on the heap is allocated (typically 2x the capacity).
2.  All elements are copied/moved to the new buffer.
3.  The old buffer is deallocated.

**Implication:** If you know the size upfront, use `Vec::with_capacity(n)` to avoid repeated reallocations.

### Iterating

```rust
let v = vec![10, 20, 30];

// Immutable iteration
for i in &v {
    println!("{}", i);
}

// Mutable iteration
let mut v = vec![10, 20, 30];
for i in &mut v {
    *i += 50; // Dereference to modify
}
```

---

## Strings (`String` vs `&str`)

Strings in Rust are often confusing for newcomers because there are two main types.

| Type | What is it? | Ownership | Mutability |
| :--- | :---------- | :-------- | :--------- |
| `String` | A growable, heap-allocated UTF-8 string. | Owned | Mutable |
| `&str` | A **string slice**. An immutable view into a `String` or a string literal. | Borrowed | Immutable |

### Creating Strings

```rust
// From a string literal (&str)
let s1 = String::from("hello");
let s2 = "hello".to_string();

// Concatenation
let s3 = s1 + " world"; // Note: s1 is MOVED here, can't be used again.

// Using format! macro (doesn't take ownership)
let s1 = String::from("Hello");
let s2 = String::from("World");
let s3 = format!("{} {}", s1, s2);
```

### Why no Indexing (`s[0]`)?

Rust strings are UTF-8 encoded. A single character (like an emoji 😀) can be 1-4 bytes. `s[0]` is ambiguous: does it mean the first *byte* or the first *character*?

```rust
let hello = "Здравствуйте"; // Russian "Hello"
// println!("{}", hello[0]); // COMPILE ERROR!

// To iterate over characters:
for c in hello.chars() {
    println!("{}", c);
}
```

---

## HashMaps (`HashMap<K, V>`)

A hash map stores key-value pairs. It's Rust's equivalent to `std::unordered_map` in C++, `HashMap` in Java, or a Python `dict`.

### Creating and Inserting

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

### Accessing Values

```rust
let team_name = String::from("Blue");

// .get() returns Option<&V>
let score = scores.get(&team_name);

match score {
    Some(s) => println!("Score: {}", s),
    None => println!("Team not found"),
}
```

### Ownership with HashMaps

For types that implement `Copy` (like `i32`), values are copied into the map. For owned types (like `String`), the map **takes ownership**.

```rust
let field_name = String::from("Color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);

// println!("{}", field_name); // COMPILE ERROR! field_name was moved.
```

### Updating a HashMap

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

// Overwriting
scores.insert(String::from("Blue"), 25);

// Only inserting if key doesn't exist
scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50); // Blue already exists, won't change

// Updating based on old value
let text = "hello world wonderful world";
let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1; // Dereference to modify
}
// map is now {"hello": 1, "world": 2, "wonderful": 1}
```
