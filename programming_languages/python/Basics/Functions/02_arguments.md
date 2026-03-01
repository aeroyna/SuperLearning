# Arguments and Parameters

## 1. Parameters vs Arguments

- **Parameters**: Variables in the function definition
- **Arguments**: Values passed when calling the function

```python
def greet(name):  # 'name' is a parameter
    return f"Hello, {name}!"

greet("Alice")  # "Alice" is an argument
```

---

## 2. Positional Arguments

Arguments matched by position:

```python
def describe(name, age, city):
    return f"{name} is {age} years old and lives in {city}"

describe("Alice", 30, "NYC")
# "Alice is 30 years old and lives in NYC"

# Order matters!
describe(30, "NYC", "Alice")  # Wrong result
```

---

## 3. Keyword Arguments

Arguments matched by name:

```python
def describe(name, age, city):
    return f"{name} is {age} years old and lives in {city}"

# All keyword arguments
describe(name="Alice", age=30, city="NYC")

# Mixed (positional first, then keyword)
describe("Alice", city="NYC", age=30)

# Error: positional after keyword
# describe(name="Alice", 30, "NYC")  # SyntaxError
```

---

## 4. Default Parameter Values

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")            # "Hello, Alice!"
greet("Alice", "Hi")      # "Hi, Alice!"
greet("Alice", greeting="Hey")  # "Hey, Alice!"
```

### Default Parameters Must Come Last
```python
# Valid
def func(a, b, c=3):
    pass

# Invalid
# def func(a, b=2, c):  # SyntaxError
#     pass
```

### Mutable Default Argument Trap
```python
# WRONG: mutable default
def append_to(item, lst=[]):
    lst.append(item)
    return lst

append_to(1)  # [1]
append_to(2)  # [1, 2] — Wrong! Same list

# CORRECT: use None
def append_to(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

### Why This Happens
Default values are evaluated **once** when the function is defined, not each time it's called:

```python
def func(lst=[]):
    print(id(lst))  # Same id every time!
    return lst

func()  # Creates list once
func()  # Reuses same list
```

---

## 5. Arbitrary Positional Arguments (`*args`)

Collect extra positional arguments into a tuple:

```python
def sum_all(*args):
    print(f"args = {args}")  # Tuple
    return sum(args)

sum_all(1, 2, 3)      # args = (1, 2, 3), returns 6
sum_all(1, 2, 3, 4, 5)  # args = (1, 2, 3, 4, 5), returns 15
```

### Combining with Regular Parameters
```python
def greet(greeting, *names):
    for name in names:
        print(f"{greeting}, {name}!")

greet("Hello", "Alice", "Bob", "Charlie")
# Hello, Alice!
# Hello, Bob!
# Hello, Charlie!
```

---

## 6. Arbitrary Keyword Arguments (`**kwargs`)

Collect extra keyword arguments into a dictionary:

```python
def print_info(**kwargs):
    print(f"kwargs = {kwargs}")  # Dict
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=30, city="NYC")
# kwargs = {'name': 'Alice', 'age': 30, 'city': 'NYC'}
# name: Alice
# age: 30
# city: NYC
```

### Combining with Regular Parameters
```python
def create_user(name, email, **extra):
    user = {"name": name, "email": email}
    user.update(extra)
    return user

create_user("Alice", "alice@example.com", age=30, city="NYC")
# {'name': 'Alice', 'email': 'alice@example.com', 'age': 30, 'city': 'NYC'}
```

---

## 7. Combining All Argument Types

Order must be:
1. Positional parameters
2. `*args`
3. Keyword-only parameters
4. `**kwargs`

```python
def func(pos1, pos2, *args, kw_only, **kwargs):
    print(f"pos1={pos1}, pos2={pos2}")
    print(f"args={args}")
    print(f"kw_only={kw_only}")
    print(f"kwargs={kwargs}")

func(1, 2, 3, 4, kw_only="required", extra="value")
# pos1=1, pos2=2
# args=(3, 4)
# kw_only=required
# kwargs={'extra': 'value'}
```

---

## 8. Keyword-Only Arguments

Arguments that MUST be passed by keyword (after `*`):

```python
def greet(name, *, greeting="Hello", punctuation="!"):
    return f"{greeting}, {name}{punctuation}"

greet("Alice")  # "Hello, Alice!"
greet("Alice", greeting="Hi")  # "Hi, Alice!"
greet("Alice", "Hi")  # TypeError: takes 1 positional argument
```

### With `*args`
```python
def func(*args, keyword_only):
    print(args, keyword_only)

func(1, 2, 3, keyword_only="value")
```

---

## 9. Positional-Only Arguments (Python 3.8+)

Arguments that MUST be passed by position (before `/`):

```python
def greet(name, /, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")           # OK
greet("Alice", "Hi")     # OK
# greet(name="Alice")    # TypeError: positional-only
```

### Full Syntax
```python
def func(pos_only, /, pos_or_kw, *, kw_only):
    pass

func(1, 2, kw_only=3)       # OK
func(1, pos_or_kw=2, kw_only=3)  # OK
# func(pos_only=1, ...)     # Error
# func(1, 2, 3)             # Error (kw_only must be keyword)
```

---

## 10. Unpacking Arguments

### Unpack List/Tuple with `*`
```python
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
add(*numbers)  # Same as add(1, 2, 3)

# With ranges
args = range(3)
add(*args)  # Same as add(0, 1, 2)
```

### Unpack Dictionary with `**`
```python
def greet(name, greeting, punctuation):
    return f"{greeting}, {name}{punctuation}"

kwargs = {"name": "Alice", "greeting": "Hello", "punctuation": "!"}
greet(**kwargs)  # Same as greet(name="Alice", greeting="Hello", punctuation="!")
```

### Combining Unpacking
```python
def func(a, b, c, d, e):
    return a + b + c + d + e

args = [1, 2]
kwargs = {"d": 4, "e": 5}
func(*args, 3, **kwargs)  # func(1, 2, 3, d=4, e=5)
```

---

## 11. Pass by Object Reference

Python uses "pass by object reference":
- The function receives a reference to the object
- Mutable objects can be modified
- Reassigning the parameter doesn't affect the original

```python
# Mutable object (list)
def modify_list(lst):
    lst.append(4)  # Modifies original
    lst = [10, 20]  # Reassigns local variable only

my_list = [1, 2, 3]
modify_list(my_list)
print(my_list)  # [1, 2, 3, 4]

# Immutable object (int)
def modify_int(x):
    x += 1  # Creates new int, doesn't affect original

my_int = 5
modify_int(my_int)
print(my_int)  # 5
```

---

## 12. Common Patterns

### Optional Parameters
```python
def connect(host, port=None, timeout=None):
    if port is None:
        port = 80
    if timeout is None:
        timeout = 30
    return f"Connecting to {host}:{port} with {timeout}s timeout"
```

### Builder Pattern with `**kwargs`
```python
def create_element(tag, text="", **attrs):
    attr_str = " ".join(f'{k}="{v}"' for k, v in attrs.items())
    return f"<{tag} {attr_str}>{text}</{tag}>"

create_element("a", "Click here", href="https://example.com", target="_blank")
# '<a href="https://example.com" target="_blank">Click here</a>'
```

### Forwarding Arguments
```python
def wrapper(*args, **kwargs):
    print(f"Calling with {args}, {kwargs}")
    return original_function(*args, **kwargs)
```

---

## 13. Practice Problems

1. Write a function that accepts any number of strings and returns them joined with a separator (default: comma).

2. Write a function `make_tag(tag, text, **attrs)` that creates an HTML tag.

3. Write a function that calculates the average of any number of numbers, with an optional `weights` keyword argument.

---

## Next Steps
- [Scope and Closures](03_scope_and_closures.md)
