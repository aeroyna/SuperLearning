# String Formatting

Python offers multiple ways to format strings. f-strings (formatted string literals) are the most modern and recommended approach.

---

## 1. f-strings (Python 3.6+)

### Basic Usage
```python
name = "Alice"
age = 30

f"My name is {name} and I'm {age} years old."
# "My name is Alice and I'm 30 years old."

# Expressions
f"Next year I'll be {age + 1}."
# "Next year I'll be 31."

# Method calls
f"Hello, {name.upper()}!"
# "Hello, ALICE!"
```

### Format Specifiers
```python
# General syntax: {value:format_spec}

# Width and alignment
f"{'left':<10}"    # "left      "
f"{'right':>10}"   # "     right"
f"{'center':^10}"  # "  center  "
f"{'fill':*^10}"   # "***fill***"

# Numbers
n = 42
f"{n:05d}"         # "00042" (zero-padded)
f"{n:+d}"          # "+42" (always show sign)
f"{n: d}"          # " 42" (space for positive)

# Floats
pi = 3.14159265
f"{pi:.2f}"        # "3.14" (2 decimal places)
f"{pi:10.2f}"      # "      3.14" (width 10)
f"{pi:.2e}"        # "3.14e+00" (scientific)
f"{pi:.2%}"        # "314.16%" (percentage, *100)

# Large numbers
big = 1234567890
f"{big:,}"         # "1,234,567,890" (thousands separator)
f"{big:_}"         # "1_234_567_890" (underscore separator)
```

### Special Formats
```python
# Binary, octal, hex
n = 255
f"{n:b}"    # "11111111" (binary)
f"{n:o}"    # "377" (octal)
f"{n:x}"    # "ff" (hex lowercase)
f"{n:X}"    # "FF" (hex uppercase)
f"{n:#x}"   # "0xff" (with prefix)

# Debug format (Python 3.8+)
name = "Alice"
f"{name=}"  # "name='Alice'"
f"{2+2=}"   # "2+2=4"
f"{name=!r}"  # "name='Alice'" (repr)
```

### Multiline f-strings
```python
name = "Alice"
age = 30

message = f"""
Name: {name}
Age: {age}
Status: {'Adult' if age >= 18 else 'Minor'}
"""
```

### Escaping Braces
```python
f"Use {{braces}} in f-strings"
# "Use {braces} in f-strings"

f"Dict literal: {{'a': 1}}"
# "Dict literal: {'a': 1}"
```

---

## 2. str.format() Method

```python
# Positional arguments
"Hello, {}!".format("World")
# "Hello, World!"

"{} + {} = {}".format(1, 2, 3)
# "1 + 2 = 3"

# Indexed arguments
"{0} vs {1}".format("Python", "JavaScript")
# "Python vs JavaScript"

"{0} is better than {0}... wait".format("Python")
# "Python is better than Python... wait"

# Named arguments
"Hello, {name}!".format(name="World")
# "Hello, World!"

# Access attributes and items
"{0.real} + {0.imag}i".format(3+4j)
# "3.0 + 4.0i"

"{person[name]} is {person[age]}".format(person={"name": "Alice", "age": 30})
# "Alice is 30"
```

### Format Specifiers (same as f-strings)
```python
"{:>10}".format("right")  # "     right"
"{:.2f}".format(3.14159)  # "3.14"
"{:,}".format(1000000)    # "1,000,000"
```

---

## 3. % Operator (Old Style)

```python
# String substitution
"Hello, %s!" % "World"
# "Hello, World!"

# Multiple values (tuple)
"Name: %s, Age: %d" % ("Alice", 30)
# "Name: Alice, Age: 30"

# Named values (dict)
"%(name)s is %(age)d" % {"name": "Alice", "age": 30}
# "Alice is 30"

# Format specifiers
"%10s" % "right"      # "     right" (width 10)
"%-10s" % "left"      # "left      " (left-aligned)
"%.2f" % 3.14159      # "3.14" (2 decimal places)
"%05d" % 42           # "00042" (zero-padded)
```

### Format Codes
| Code | Type |
|------|------|
| `%s` | String |
| `%d` | Integer |
| `%f` | Float |
| `%x` | Hex |
| `%o` | Octal |
| `%%` | Literal % |

---

## 4. Template Strings

```python
from string import Template

t = Template("Hello, $name!")
t.substitute(name="World")  # "Hello, World!"

# Safe substitution (missing keys don't raise errors)
t = Template("$name is $age years old")
t.safe_substitute(name="Alice")
# "Alice is $age years old"

# Escape $
Template("Price: $$100").substitute()  # "Price: $100"
```

---

## 5. Common Formatting Patterns

### Table Formatting
```python
data = [
    ("Alice", 30, 50000),
    ("Bob", 25, 45000),
    ("Charlie", 35, 60000),
]

print(f"{'Name':<10} {'Age':>5} {'Salary':>10}")
print("-" * 27)
for name, age, salary in data:
    print(f"{name:<10} {age:>5} {salary:>10,}")

# Name        Age     Salary
# ---------------------------
# Alice         30     50,000
# Bob           25     45,000
# Charlie       35     60,000
```

### Currency Formatting
```python
amount = 1234567.89

f"${amount:,.2f}"           # "$1,234,567.89"
f"${amount:>15,.2f}"        # "  $1,234,567.89"

import locale
locale.setlocale(locale.LC_ALL, 'en_US.UTF-8')
locale.currency(amount, grouping=True)  # "$1,234,567.89"
```

### Date and Time
```python
from datetime import datetime

now = datetime.now()

f"{now:%Y-%m-%d}"           # "2024-01-15"
f"{now:%H:%M:%S}"           # "14:30:45"
f"{now:%B %d, %Y}"          # "January 15, 2024"
f"{now:%A}"                 # "Monday"
```

### Progress Bar
```python
def progress_bar(progress, total, width=50):
    filled = int(width * progress / total)
    bar = "█" * filled + "░" * (width - filled)
    percent = progress / total * 100
    return f"\r[{bar}] {percent:.1f}%"

print(progress_bar(75, 100))
# [█████████████████████████████████████░░░░░░░░░░░░░] 75.0%
```

---

## 6. Format Specification Mini-Language

Full syntax: `[[fill]align][sign][#][0][width][grouping][.precision][type]`

| Component | Description |
|-----------|-------------|
| `fill` | Any character (default: space) |
| `align` | `<` left, `>` right, `^` center, `=` pad after sign |
| `sign` | `+` always, `-` negative only, ` ` space for positive |
| `#` | Alternate form (0x for hex, etc.) |
| `0` | Zero-padding |
| `width` | Minimum field width |
| `grouping` | `,` or `_` for thousands |
| `precision` | Decimal places or max string length |
| `type` | `d`, `f`, `e`, `s`, `b`, `x`, etc. |

```python
# Examples
f"{42:+010d}"      # "+000000042" (sign, zero-pad, width 10, decimal)
f"{3.14:+.2f}"     # "+3.14" (always show sign, 2 decimals)
f"{255:#08x}"      # "0x0000ff" (hex with prefix, zero-pad width 8)
f"{1000000:,.2f}"  # "1,000,000.00" (thousands, 2 decimals)
```

---

## 7. When to Use Which

| Method | Use When |
|--------|----------|
| f-strings | Modern code, best readability |
| .format() | Python 2 compatibility, complex templates |
| % operator | Legacy code, simple substitution |
| Template | User-provided templates (security) |

**Recommendation**: Use f-strings for most cases.

---

## 8. Practice Problems

1. Format a phone number: `1234567890` → `(123) 456-7890`

2. Create a function that prints a centered, boxed message.

3. Format a list of items as a bulleted list with aligned prices.

4. Create a simple table formatter that auto-adjusts column widths.
