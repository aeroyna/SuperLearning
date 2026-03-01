# Operator Overloading

Operator overloading is a compile-time polymorphism feature in C++ that allows you to redefine the behavior of operators for user-defined types (classes and structs). This makes your custom types feel like built-in types.

## Why Operator Overloading?

Consider a `Complex` number class. Without operator overloading:
```cpp
Complex c1(3, 4), c2(1, 2);
Complex c3 = c1.add(c2);  // awkward
```

With operator overloading:
```cpp
Complex c3 = c1 + c2;  // natural and intuitive
```

## Syntax

Operators are overloaded by defining a special function with the keyword `operator` followed by the operator symbol.

```cpp
ReturnType operator symbol (parameters) {
    // implementation
}
```

## Operators That Can Be Overloaded

| Category | Operators |
|----------|-----------|
| Arithmetic | `+` `-` `*` `/` `%` |
| Relational | `==` `!=` `<` `>` `<=` `>=` |
| Logical | `&&` `\|\|` `!` |
| Bitwise | `&` `\|` `^` `~` `<<` `>>` |
| Assignment | `=` `+=` `-=` `*=` `/=` etc. |
| Increment/Decrement | `++` `--` |
| Subscript | `[]` |
| Function Call | `()` |
| Member Access | `->` `->*` |
| Memory | `new` `delete` `new[]` `delete[]` |
| Stream | `<<` `>>` |
| Other | `,` `&` `*` |

## Operators That CANNOT Be Overloaded

- `::` (scope resolution)
- `.` (member access)
- `.*` (member pointer access)
- `?:` (ternary conditional)
- `sizeof`
- `typeid`
- `alignof`

## Member vs Non-Member Overloading

Operators can be overloaded as:
1. **Member functions** - left operand is `this`
2. **Non-member functions** - often declared as `friend`

```cpp
// Member function: a + b calls a.operator+(b)
Complex Complex::operator+(const Complex& rhs) const;

// Non-member function: a + b calls operator+(a, b)
Complex operator+(const Complex& lhs, const Complex& rhs);
```

## In This Chapter

- [[01_arithmetic_operators|Arithmetic Operators]]
- [[02_comparison_operators|Comparison Operators]]
- [[03_stream_operators|Stream Operators (<<, >>)]]
- [[04_assignment_operator|Assignment Operator]]
- [[05_subscript_and_function_call|Subscript and Function Call Operators]]
- [[06_increment_decrement|Increment and Decrement Operators]]
- [[practice_problems|Practice Problems]]

## Key Takeaways

- Operator overloading makes user-defined types intuitive to use
- Cannot change operator precedence or associativity
- Cannot create new operators
- At least one operand must be a user-defined type
- Some operators (`=`, `[]`, `()`, `->`) must be member functions

## Common Interview Questions

> [!question]- Why can't `.` and `::` be overloaded?
> These operators are fundamental to the language's syntax and semantics. Overloading them would make code ambiguous and break the ability to access actual members.

> [!question]- When should you use member vs non-member overloading?
> Use member when the left operand is always your type. Use non-member (friend) when you need implicit conversion on the left operand, or for symmetric operations like `+`.
