# Template Specialization

Template specialization allows you to provide a different implementation for a template when it is instantiated with a specific type.

## Why specialize a template?

Sometimes, the generic implementation of a template may not be suitable for all types. For example:

*   The generic implementation might not be efficient for a particular type.
*   The generic implementation might not compile for a particular type (e.g., if it uses an operator that the type doesn't support).

## Function Template Specialization

You can provide a specialized version of a function template for a specific type.

### Example

Let's consider a `swap` function template. The generic version uses a temporary variable.

```cpp
template <typename T>
void swap_val(T& a, T& b) {
    T temp = a;
    a = b;
    b = temp;
}
```

Now, let's say we have a `Widget` class that contains a large amount of data. Swapping two `Widget` objects using the generic `swap_val` would be inefficient because it would involve copying the large data. We can provide a specialized version of `swap_val` for `Widget` that is more efficient (e.g., by swapping pointers to the data).

```cpp
class Widget { /* ... large data ... */ };

// Template specialization for Widget
template <>
void swap_val<Widget>(Widget& a, Widget& b) {
    // efficient swap implementation for Widget
}
```

## Class Template Specialization

You can also specialize a class template.

### Example

Let's say we have a `MyContainer` class template.

```cpp
template <typename T>
class MyContainer {
    // generic implementation
};
```

We can provide a specialized version for `bool`. `std::vector<bool>` is a well-known example of this, where it is specialized to use bits instead of bytes to store the booleans, making it more memory-efficient.

```cpp
// Class template specialization for bool
template <>
class MyContainer<bool> {
    // specialized implementation for bool
};
```

## Partial Template Specialization

You can also partially specialize a template. This is where you specialize a template for a certain category of types, but not for a single specific type.

For example, you can specialize a template for all pointer types.

```cpp
// Generic template
template <typename T>
class MyClass { ... };

// Partial specialization for pointer types
template <typename T>
class MyClass<T*> {
    // implementation for pointers
};
```
Now, `MyClass<int>` would use the generic template, but `MyClass<int*>` would use the partially specialized template.

Template specialization is a powerful feature that gives you fine-grained control over how your templates behave with different types.
