# Lifetimes

Lifetimes are Rust's way of ensuring that **references are always valid**. They are annotations that tell the compiler how long a reference should live. This prevents **dangling pointers** at compile time.

**Key Insight:** Lifetimes don't *change* how long data lives. They help the compiler *verify* that references don't outlive the data they point to.

---

## The Problem Lifetimes Solve

Consider this function that returns the longest of two string slices:

```rust
// THIS WON'T COMPILE!
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

**Why the error?** The compiler doesn't know if the returned reference will be valid. It could point to `x` or `y`. If the data that `x` or `y` points to goes out of scope before the returned reference is used, you'd have a dangling pointer.

---

## Lifetime Annotations

Lifetime annotations create a relationship between the lifetimes of the parameters and the return value.

```rust
// The lifetime 'a says:
// "The returned reference will be valid as long as BOTH x and y are valid."
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

### `'a` is a Constraint, Not a Duration

The annotation `'a` doesn't mean "x and y live for the same amount of time." It means "the borrow checker should ensure the returned reference doesn't outlive the *shorter* of the two input lifetimes."

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
        // result is valid here because string2 is still in scope.
        println!("The longest string is {}", result);
    }
    // println!("{}", result); // COMPILE ERROR if this line were uncommented!
    // result *might* point to string2, which is now out of scope.
}
```

---

## Lifetime Elision

The compiler can often **infer** lifetimes, so you don't have to write them explicitly. This is called **lifetime elision**.

### The Elision Rules

1.  Each parameter that is a reference gets its own lifetime parameter.
2.  If there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters.
3.  If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self`, the lifetime of `self` is assigned to all output lifetime parameters.

```rust
// What you write (elided):
fn first_word(s: &str) -> &str { ... }

// What the compiler infers:
fn first_word<'a>(s: &'a str) -> &'a str { ... }
```

---

## The `'static` Lifetime

The `'static` lifetime is special. It means the reference can live for the **entire duration of the program**.

All string literals have a `'static` lifetime because they are baked into the program's binary.

```rust
let s: &'static str = "I have a static lifetime.";
```

### Common Misuse

Avoid overusing `'static`. It's a strong constraint. If the compiler suggests adding `'static`, it's often a sign that you should refactor your code to not require such a long-lived reference (e.g., by cloning data or restructuring ownership).

---

## Lifetimes in Structs

If a struct holds a reference, it needs a lifetime annotation. This tells the compiler that the struct cannot outlive the data it references.

```rust
struct ImportantExcerpt<'a> {
    part: &'a str, // This struct holds a reference
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    
    let i = ImportantExcerpt {
        part: first_sentence,
    };
    // `i` cannot outlive `novel`.
}
```
