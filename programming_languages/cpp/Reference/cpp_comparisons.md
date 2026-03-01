# C++ "X vs Y" Comparisons

Common "what's the difference" interview questions answered concisely.

## Pointers vs References

| Aspect | Pointer | Reference |
|--------|---------|-----------|
| Can be null | Yes | No |
| Can be reassigned | Yes | No |
| Can be uninitialized | Yes | No |
| Syntax | `*ptr`, `ptr->` | Direct access |
| Memory | Takes space | May be optimized away |

**Use reference when**: You don't need null or reassignment.

## `const` Pointer Variations

```cpp
const int* p;        // Pointer to const int (can't modify *p)
int* const p;        // Const pointer to int (can't modify p)
const int* const p;  // Both const
```

## `new`/`delete` vs `malloc`/`free`

| Aspect | new/delete | malloc/free |
|--------|------------|-------------|
| Calls constructor | Yes | No |
| Type safe | Yes | No (returns void*) |
| Size | Automatic | Manual |
| Can be overloaded | Yes | No |
| Exception on fail | Yes | Returns NULL |

## `++i` vs `i++`

| Aspect | `++i` (prefix) | `i++` (postfix) |
|--------|---------------|-----------------|
| Return value | New value | Old value |
| Efficiency | Better for iterators | Creates temporary |

**Prefer `++i`** unless you need the old value.

## Array vs Vector

| Aspect | `std::array<T,N>` | `std::vector<T>` |
|--------|-------------------|------------------|
| Size | Fixed at compile time | Dynamic |
| Memory | Stack (usually) | Heap |
| Overhead | None | Small (capacity tracking) |
| C array compatible | Yes (`.data()`) | Yes (`.data()`) |

## map vs unordered_map

| Aspect | `std::map` | `std::unordered_map` |
|--------|------------|---------------------|
| Order | Sorted by key | No order |
| Complexity | O(log n) | O(1) average |
| Implementation | Red-black tree | Hash table |
| Key requirement | `operator<` | `std::hash` + `==` |

**Use map** when you need sorted order or range queries.

## set vs unordered_set

Same as map vs unordered_map, but stores only keys.

## shared_ptr vs unique_ptr

| Aspect | `unique_ptr` | `shared_ptr` |
|--------|--------------|--------------|
| Ownership | Exclusive | Shared |
| Copying | No (move only) | Yes |
| Overhead | None | Reference counting |
| Circular refs | N/A | Possible (use weak_ptr) |

**Default to `unique_ptr`**, use `shared_ptr` when sharing is needed.

## virtual vs non-virtual

| Aspect | Non-virtual | Virtual |
|--------|-------------|---------|
| Dispatch | Static (compile time) | Dynamic (runtime) |
| Overhead | None | vtable lookup |
| Override | Hides, doesn't override | True polymorphism |
| Use case | No inheritance | Polymorphic behavior |

## override vs final

| Keyword | Purpose |
|---------|---------|
| `override` | Ensure function overrides base virtual |
| `final` | Prevent further override (or inheritance) |

## `const` vs `constexpr`

| Aspect | `const` | `constexpr` |
|--------|---------|-------------|
| When evaluated | Runtime possible | Must be compile time |
| Variables | Immutable | Compile-time constant |
| Functions | Can't use on functions | Must evaluate at compile time |

## `#define` vs `const`

| Aspect | `#define` | `const` |
|--------|-----------|---------|
| Type safety | No | Yes |
| Scope | Global | Respects scope |
| Debugging | Symbol lost | Symbol preserved |
| Can be pointer | No | Yes |

**Prefer `const` or `constexpr`** in C++.

## throw vs noexcept

| Aspect | Can throw | `noexcept` |
|--------|-----------|------------|
| Exceptions | Allowed | Calls `std::terminate` if thrown |
| Optimization | Less | More (compiler knows) |
| Move ops | Should avoid throwing | Mark noexcept for optimization |

## Copy vs Move

| Aspect | Copy | Move |
|--------|------|------|
| Source | Unchanged | Left in valid but unspecified state |
| Cost | Allocates new resources | Just transfers ownership |
| When used | Lvalue | Rvalue (or `std::move`) |

## class vs typename (in templates)

Mostly interchangeable. Convention:
- `typename` for generic "any type"
- `class` when expecting a class type

`typename` is required for dependent types:
```cpp
typename Container::iterator it;  // Must use typename here
```

## `=` vs `==` vs `===`

C++ only has `=` (assignment) and `==` (comparison). No `===`.

## NULL vs nullptr

| Aspect | `NULL` | `nullptr` |
|--------|--------|-----------|
| Type | `int` (0) | `std::nullptr_t` |
| Ambiguity | Yes (int vs pointer) | No |

**Always use `nullptr`** in C++11+.

## struct vs class

Only difference: default access.
- `struct`: public by default
- `class`: private by default

Convention: struct for POD, class for objects with behavior.

## Inheritance: public vs protected vs private

| Base member | public inheritance | protected | private |
|-------------|-------------------|-----------|---------|
| public | public | protected | private |
| protected | protected | protected | private |
| private | (inaccessible) | (inaccessible) | (inaccessible) |
