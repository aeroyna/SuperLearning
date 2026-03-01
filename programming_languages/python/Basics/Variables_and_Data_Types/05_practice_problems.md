# Practice Problems: Variables and Data Types

## Easy

### 1. Type Detective
Predict the type of each expression:

```python
a = 3 + 4.0
b = 10 // 3
c = 10 / 3
d = True + True + True
e = "hello" * 3
f = [1, 2] + [3, 4]
```

<details>
<summary>Solution</summary>

```python
type(a)  # float — int + float = float
type(b)  # int — floor division returns int
type(c)  # float — true division always returns float
type(d)  # int — True=1, so 1+1+1=3 (int, not bool)
type(e)  # str — string repetition
type(f)  # list — list concatenation
```
</details>

---

### 2. Variable Swap
Swap two variables without using a temporary variable:

```python
x = 10
y = 20
# Your code here
print(x, y)  # Should print: 20 10
```

<details>
<summary>Solution</summary>

```python
x, y = y, x
```
</details>

---

### 3. None Check
Write a function that returns `"No value"` if the input is `None`, otherwise returns the input doubled.

```python
def double_or_none(value):
    # Your code here
    pass

print(double_or_none(5))     # 10
print(double_or_none(None))  # "No value"
print(double_or_none("hi"))  # "hihi"
```

<details>
<summary>Solution</summary>

```python
def double_or_none(value):
    if value is None:
        return "No value"
    return value * 2
```
</details>

---

## Medium

### 4. Reference Puzzle
Predict the output:

```python
a = [1, 2, 3]
b = a
c = a[:]
a.append(4)
b.append(5)
c.append(6)

print(a)
print(b)
print(c)
```

<details>
<summary>Solution</summary>

```python
print(a)  # [1, 2, 3, 4, 5]
print(b)  # [1, 2, 3, 4, 5]  — same object as a
print(c)  # [1, 2, 3, 6]     — different object (shallow copy)
```
</details>

---

### 5. Type Coercion Chain
What's the final type and value?

```python
result = True + 3.14 + 2j
print(type(result), result)
```

<details>
<summary>Solution</summary>

```python
# Step 1: True + 3.14 → 1 + 3.14 → 4.14 (float)
# Step 2: 4.14 + 2j → (4.14+2j) (complex)
print(type(result), result)  # <class 'complex'> (4.140000000000001+2j)
```
</details>

---

### 6. Safe Division
Write a function that performs division but returns `None` for division by zero and converts string numbers:

```python
def safe_divide(a, b):
    # Your code here
    pass

print(safe_divide(10, 2))      # 5.0
print(safe_divide(10, 0))      # None
print(safe_divide("10", "2"))  # 5.0
print(safe_divide("10", "a"))  # None
```

<details>
<summary>Solution</summary>

```python
def safe_divide(a, b):
    try:
        a = float(a)
        b = float(b)
        if b == 0:
            return None
        return a / b
    except (ValueError, TypeError):
        return None
```
</details>

---

### 7. Mutable Default Trap
Fix this buggy function:

```python
def add_item(item, items=[]):
    items.append(item)
    return items

# Bug demonstration:
print(add_item(1))  # [1]
print(add_item(2))  # [1, 2] — Wrong! Should be [2]
print(add_item(3))  # [1, 2, 3] — Wrong! Should be [3]
```

<details>
<summary>Solution</summary>

```python
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```
</details>

---

## Hard

### 8. Integer Interning
Explain the output of each comparison:

```python
a = 256
b = 256
print(f"256: {a is b}")

c = 257
d = 257
print(f"257: {c is d}")

e = 257; f = 257
print(f"257 same line: {e is f}")

g = int("257")
h = int("257")
print(f"int('257'): {g is h}")
```

<details>
<summary>Solution</summary>

```python
# 256: True — integers -5 to 256 are cached
# 257: False — outside cache, different objects
# 257 same line: True — compiler optimization (same constant)
# int("257"): False — runtime conversion, no caching
```
</details>

---

### 9. Nested Mutability
Predict and explain:

```python
original = [[1, 2], [3, 4]]
shallow = original[:]
deep = [row[:] for row in original]

original[0].append(5)
original.append([6, 7])

print("original:", original)
print("shallow:", shallow)
print("deep:", deep)
```

<details>
<summary>Solution</summary>

```python
print("original:", original)  # [[1, 2, 5], [3, 4], [6, 7]]
print("shallow:", shallow)    # [[1, 2, 5], [3, 4]]
print("deep:", deep)          # [[1, 2], [3, 4]]

# Explanation:
# - shallow copy creates new outer list but shares inner lists
# - original[0].append(5) affects shallow because they share [1,2]
# - original.append([6,7]) doesn't affect shallow (different outer list)
# - deep copy creates new inner lists too, completely independent
```
</details>

---

### 10. Type Hierarchy Challenge
Write a function that determines the "most specific common type" of two values:

```python
def common_type(a, b):
    """
    Returns the most specific common type.
    Examples:
    - common_type(1, 2) → int
    - common_type(1, 1.5) → float (int is subtype of float in numeric tower)
    - common_type(1, "a") → object
    - common_type(True, 1) → int (bool is subtype of int)
    """
    # Your code here
    pass
```

<details>
<summary>Solution</summary>

```python
def common_type(a, b):
    type_a = type(a)
    type_b = type(b)

    # Check if one is subtype of other
    if isinstance(a, type_b):
        return type_b
    if isinstance(b, type_a):
        return type_a

    # Find common ancestor in MRO
    mro_a = type_a.__mro__
    mro_b = set(type_b.__mro__)

    for t in mro_a:
        if t in mro_b:
            return t

    return object  # Fallback

# Note: This is simplified. Real numeric tower handling
# would need special cases for int/float/complex.
```
</details>

---

### 11. Custom Truthy Class
Create a class `NonEmptyString` that:
- Is falsy when the string is empty or only whitespace
- Is truthy otherwise

```python
class NonEmptyString:
    # Your code here
    pass

s1 = NonEmptyString("hello")
s2 = NonEmptyString("")
s3 = NonEmptyString("   ")

print(bool(s1))  # True
print(bool(s2))  # False
print(bool(s3))  # False

if s1:
    print("s1 has content")  # Should print
```

<details>
<summary>Solution</summary>

```python
class NonEmptyString:
    def __init__(self, value):
        self.value = value

    def __bool__(self):
        return bool(self.value.strip())

    def __repr__(self):
        return f"NonEmptyString({self.value!r})"
```
</details>

---

## Challenge Problem

### 12. Memory Inspector
Write a function that shows the memory relationships between objects:

```python
def inspect_memory(*objects, names=None):
    """
    Print memory analysis of objects showing:
    - Type
    - ID
    - Which objects share the same ID
    """
    # Your code here

# Usage:
a = [1, 2, 3]
b = a
c = [1, 2, 3]
d = a[:]

inspect_memory(a, b, c, d, names=['a', 'b', 'c', 'd'])

# Expected output:
# a: list id=140...
# b: list id=140... (same as: a)
# c: list id=141...
# d: list id=142...
```

<details>
<summary>Solution</summary>

```python
def inspect_memory(*objects, names=None):
    if names is None:
        names = [f"obj{i}" for i in range(len(objects))]

    id_to_names = {}

    # First pass: collect all IDs
    for name, obj in zip(names, objects):
        obj_id = id(obj)
        if obj_id not in id_to_names:
            id_to_names[obj_id] = []
        id_to_names[obj_id].append(name)

    # Second pass: print analysis
    seen_ids = set()
    for name, obj in zip(names, objects):
        obj_id = id(obj)
        same_as = [n for n in id_to_names[obj_id] if n != name]

        same_str = f" (same as: {', '.join(same_as)})" if same_as and obj_id not in seen_ids else ""
        if obj_id in seen_ids:
            same_str = f" (same as: {id_to_names[obj_id][0]})"

        print(f"{name}: {type(obj).__name__} id={obj_id}{same_str}")
        seen_ids.add(obj_id)
```
</details>

---

## Summary Checklist

After completing these problems, you should understand:
- [ ] Dynamic vs static typing
- [ ] Strong vs weak typing
- [ ] Reference semantics in Python
- [ ] Mutable vs immutable types
- [ ] Integer caching and string interning
- [ ] Truthiness and falsiness
- [ ] The `None` type and its uses
- [ ] Type introspection functions
- [ ] Shallow vs deep copy
