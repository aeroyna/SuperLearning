# Lambda Functions

## 1. Lambda Syntax

Lambda functions are small anonymous functions:

```python
# syntax: lambda arguments: expression

# Regular function
def add(x, y):
    return x + y

# Equivalent lambda
add = lambda x, y: x + y

add(3, 5)  # 8
```

### Key Characteristics
- Single expression only (no statements)
- Implicit return
- No name (anonymous)
- Can have any number of arguments

---

## 2. Lambda Examples

### No Arguments
```python
get_pi = lambda: 3.14159
get_pi()  # 3.14159
```

### One Argument
```python
square = lambda x: x ** 2
square(5)  # 25

double = lambda x: x * 2
double(5)  # 10
```

### Multiple Arguments
```python
add = lambda x, y: x + y
add(3, 5)  # 8

power = lambda base, exp: base ** exp
power(2, 10)  # 1024
```

### Default Arguments
```python
greet = lambda name, greeting="Hello": f"{greeting}, {name}!"
greet("Alice")        # "Hello, Alice!"
greet("Alice", "Hi")  # "Hi, Alice!"
```

### `*args` and `**kwargs`
```python
sum_all = lambda *args: sum(args)
sum_all(1, 2, 3, 4)  # 10

make_dict = lambda **kwargs: kwargs
make_dict(a=1, b=2)  # {'a': 1, 'b': 2}
```

---

## 3. Lambdas with Built-in Functions

### `sorted()` — Custom Sort Key
```python
# Sort by second element
pairs = [(1, 'one'), (3, 'three'), (2, 'two')]
sorted(pairs, key=lambda x: x[1])
# [(1, 'one'), (3, 'three'), (2, 'two')]

# Sort by length
words = ['python', 'java', 'c', 'javascript']
sorted(words, key=lambda x: len(x))
# ['c', 'java', 'python', 'javascript']

# Sort objects by attribute
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

people = [Person("Alice", 30), Person("Bob", 25)]
sorted(people, key=lambda p: p.age)
```

### `map()` — Transform Elements
```python
numbers = [1, 2, 3, 4, 5]

# Square each number
list(map(lambda x: x ** 2, numbers))
# [1, 4, 9, 16, 25]

# Convert to strings
list(map(lambda x: str(x), numbers))
# ['1', '2', '3', '4', '5']

# Multiple iterables
a = [1, 2, 3]
b = [10, 20, 30]
list(map(lambda x, y: x + y, a, b))
# [11, 22, 33]
```

### `filter()` — Select Elements
```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Keep even numbers
list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4, 6, 8, 10]

# Keep positive numbers
list(filter(lambda x: x > 0, [-2, -1, 0, 1, 2]))
# [1, 2]

# Remove None values
list(filter(lambda x: x is not None, [1, None, 2, None, 3]))
# [1, 2, 3]
# Or simpler: list(filter(None, [1, None, 2, None, 3]))
```

### `reduce()` — Aggregate Elements
```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# Sum
reduce(lambda acc, x: acc + x, numbers)
# 15

# Product
reduce(lambda acc, x: acc * x, numbers)
# 120

# Max (without using max())
reduce(lambda a, b: a if a > b else b, numbers)
# 5

# With initial value
reduce(lambda acc, x: acc + x, numbers, 100)
# 115
```

---

## 4. Lambdas in Data Structures

### Dictionary Value Sorting
```python
data = {'a': 3, 'b': 1, 'c': 2}

# Sort by value
sorted(data.items(), key=lambda x: x[1])
# [('b', 1), ('c', 2), ('a', 3)]

# Sort by key length
data = {'python': 1, 'js': 2, 'go': 3}
sorted(data.items(), key=lambda x: len(x[0]))
# [('js', 2), ('go', 3), ('python', 1)]
```

### Grouping
```python
from itertools import groupby

data = [
    {'name': 'Alice', 'dept': 'Engineering'},
    {'name': 'Bob', 'dept': 'Engineering'},
    {'name': 'Charlie', 'dept': 'Sales'},
]

# Must sort before groupby
sorted_data = sorted(data, key=lambda x: x['dept'])
for dept, members in groupby(sorted_data, key=lambda x: x['dept']):
    print(f"{dept}: {[m['name'] for m in members]}")
# Engineering: ['Alice', 'Bob']
# Sales: ['Charlie']
```

---

## 5. Lambdas vs Comprehensions

Often, list comprehensions are clearer:

```python
numbers = [1, 2, 3, 4, 5]

# map + lambda
list(map(lambda x: x ** 2, numbers))

# Comprehension (preferred)
[x ** 2 for x in numbers]

# filter + lambda
list(filter(lambda x: x % 2 == 0, numbers))

# Comprehension (preferred)
[x for x in numbers if x % 2 == 0]
```

---

## 6. IIFE (Immediately Invoked Function Expression)

```python
# Define and call immediately
result = (lambda x, y: x + y)(3, 5)
# result = 8

# Useful in comprehensions
[
    (lambda n: n ** 2 if n > 0 else 0)(x)
    for x in [-2, -1, 0, 1, 2]
]
# [0, 0, 0, 1, 4]
```

---

## 7. Closure with Lambda

```python
def make_multiplier(n):
    return lambda x: x * n

double = make_multiplier(2)
triple = make_multiplier(3)

double(5)  # 10
triple(5)  # 15
```

### Late Binding Gotcha
```python
# Problem
funcs = [lambda x: x + i for i in range(5)]
[f(10) for f in funcs]  # [14, 14, 14, 14, 14] — all use i=4

# Solution: capture i as default argument
funcs = [lambda x, i=i: x + i for i in range(5)]
[f(10) for f in funcs]  # [10, 11, 12, 13, 14]
```

---

## 8. Conditional Expression in Lambda

```python
# Ternary expression
sign = lambda x: "positive" if x > 0 else "negative" if x < 0 else "zero"

sign(5)   # "positive"
sign(-3)  # "negative"
sign(0)   # "zero"

# Multiple conditions
classify = lambda x: (
    "small" if x < 10 else
    "medium" if x < 100 else
    "large"
)
```

---

## 9. When to Use Lambda

### Good Use Cases
```python
# Sort key
sorted(items, key=lambda x: x.name)

# Short callbacks
button.on_click(lambda event: print("Clicked!"))

# Simple transformations
list(map(lambda x: x.strip(), lines))
```

### When to Use Regular Functions
```python
# Complex logic
# Bad: hard to read lambda
filter(lambda x: x is not None and x > 0 and x % 2 == 0, items)

# Good: named function
def is_positive_even(x):
    if x is None:
        return False
    return x > 0 and x % 2 == 0

filter(is_positive_even, items)

# Reusable logic
def calculate_discount(price, discount_rate):
    """Apply discount and round to 2 decimal places."""
    return round(price * (1 - discount_rate), 2)
```

---

## 10. PEP 8 Guidelines

### Don't Assign Lambda to Variable
```python
# Bad
square = lambda x: x ** 2

# Good
def square(x):
    return x ** 2
```

If you need to name a function, use `def`.

### Lambda Is for One-Time Use
```python
# Good: inline use
sorted(items, key=lambda x: x.value)

# Good: as argument
map(lambda x: x * 2, numbers)
```

---

## 11. Functional Programming Tools

### `functools.partial`
```python
from functools import partial

def power(base, exponent):
    return base ** exponent

square = partial(power, exponent=2)
cube = partial(power, exponent=3)

square(5)  # 25
cube(5)    # 125
```

### `operator` Module
```python
from operator import add, mul, itemgetter, attrgetter

# Instead of lambda x, y: x + y
reduce(add, [1, 2, 3, 4])  # 10

# Instead of lambda x: x['name']
sorted(items, key=itemgetter('name'))

# Instead of lambda x: x.name
sorted(items, key=attrgetter('name'))
```

---

## 12. Practice Problems

1. Use `map` and lambda to convert a list of temperatures from Celsius to Fahrenheit.

2. Use `filter` and lambda to get all strings longer than 5 characters.

3. Use `sorted` and lambda to sort a list of dictionaries by multiple keys.

4. Implement a simple calculator using a dictionary of lambdas.

---

## Next Steps
- [Decorators Preview](05_decorators_preview.md)
