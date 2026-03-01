# Regular Expressions

Regular expressions are powerful patterns for searching, matching, and manipulating text.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Pattern Syntax**](01_pattern_syntax.md) | Characters, quantifiers, groups |
| [**2. The re Module**](02_re_module.md) | search, match, findall, sub |
| [**3. Common Patterns**](03_common_patterns.md) | Email, URL, phone, dates |

---

## Quick Reference

### Basic Patterns
```python
import re

# Match literal text
re.search(r"hello", "hello world")

# Character classes
r"[aeiou]"    # Any vowel
r"[0-9]"      # Any digit
r"[a-zA-Z]"   # Any letter
r"[^0-9]"     # Not a digit

# Shorthand classes
r"\d"         # Digit [0-9]
r"\w"         # Word char [a-zA-Z0-9_]
r"\s"         # Whitespace
r"."          # Any char (except newline)

# Quantifiers
r"a*"         # 0 or more
r"a+"         # 1 or more
r"a?"         # 0 or 1
r"a{3}"       # Exactly 3
r"a{2,4}"     # 2 to 4
```

### Common Functions
```python
import re

text = "Hello World 123"

# Search
re.search(r"\d+", text)  # <Match '123'>

# Match (start of string)
re.match(r"Hello", text)  # <Match 'Hello'>

# Find all
re.findall(r"\w+", text)  # ['Hello', 'World', '123']

# Substitute
re.sub(r"\d+", "XXX", text)  # "Hello World XXX"

# Split
re.split(r"\s+", text)  # ['Hello', 'World', '123']
```

### Groups
```python
pattern = r"(\w+)@(\w+)\.(\w+)"
match = re.search(pattern, "user@example.com")

match.group(0)  # "user@example.com" (full match)
match.group(1)  # "user"
match.group(2)  # "example"
match.group(3)  # "com"
match.groups()  # ("user", "example", "com")
```

### Named Groups
```python
pattern = r"(?P<name>\w+)@(?P<domain>\w+\.\w+)"
match = re.search(pattern, "user@example.com")

match.group("name")    # "user"
match.group("domain")  # "example.com"
match.groupdict()      # {"name": "user", "domain": "example.com"}
```

---

## Compiled Patterns

```python
# Compile for reuse
pattern = re.compile(r"\d+", re.IGNORECASE)

pattern.search(text)
pattern.findall(text)
```

---

## Flags

```python
re.IGNORECASE  # or re.I — case-insensitive
re.MULTILINE   # or re.M — ^ and $ match line boundaries
re.DOTALL      # or re.S — . matches newline
re.VERBOSE     # or re.X — allow comments and whitespace

# Combine flags
re.search(pattern, text, re.IGNORECASE | re.MULTILINE)
```

---

## Next Steps
Start with [Pattern Syntax](01_pattern_syntax.md).
