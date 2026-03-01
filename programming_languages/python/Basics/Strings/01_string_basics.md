# String Basics

## 1. Creating Strings

### String Literals
```python
# Single or double quotes
s1 = 'Hello'
s2 = "Hello"

# Triple quotes for multiline
s3 = """This is
a multiline
string"""

s4 = '''Also works
with single quotes'''

# Empty string
empty = ""
empty = str()
```

### Special Characters (Escape Sequences)
```python
s = "Hello\nWorld"     # Newline
s = "Tab\there"        # Tab
s = "Quote: \"Hi\""    # Escaped quote
s = "Backslash: \\"    # Backslash

# Common escapes
# \n  newline
# \t  tab
# \\  backslash
# \'  single quote
# \"  double quote
# \r  carriage return
# \0  null character
```

### Raw Strings
```python
# No escape processing
path = r"C:\Users\name\folder"
regex = r"\d+\.\d+"

# Useful for regex and Windows paths
import re
re.search(r"\d{3}", text)
```

### Unicode
```python
# Unicode literals
s = "Hello, 世界! 🌍"

# Unicode escape
s = "\u4e16\u754c"  # 世界
s = "\U0001F30D"    # 🌍

# Unicode name
s = "\N{LATIN SMALL LETTER A}"  # 'a'
```

---

## 2. String Operations

### Concatenation
```python
s1 = "Hello"
s2 = "World"

s1 + " " + s2      # "Hello World"
" ".join([s1, s2])  # "Hello World" (preferred for multiple)
```

### Repetition
```python
"-" * 20           # "--------------------"
"abc" * 3          # "abcabcabc"
```

### Length
```python
len("Hello")       # 5
len("世界")        # 2 (characters, not bytes)
```

### Membership
```python
"ell" in "Hello"       # True
"world" not in "Hello"  # True
```

---

## 3. Indexing and Slicing

### Indexing
```python
s = "Hello, World!"

# Positive index (from start)
s[0]    # 'H'
s[1]    # 'e'

# Negative index (from end)
s[-1]   # '!'
s[-2]   # 'd'
```

### Slicing
```python
s = "Hello, World!"

# [start:stop] — stop is exclusive
s[0:5]    # "Hello"
s[7:12]   # "World"

# Omitting indices
s[:5]     # "Hello" (from start)
s[7:]     # "World!" (to end)
s[:]      # "Hello, World!" (full copy)

# Step
s[::2]    # "Hlo ol!" (every 2nd char)
s[::-1]   # "!dlroW ,olleH" (reversed)

# Negative indices in slices
s[-6:-1]  # "World"
s[-6:]    # "World!"
```

---

## 4. String Comparison

```python
# Equality
"hello" == "hello"  # True
"hello" == "Hello"  # False (case-sensitive)

# Case-insensitive comparison
"hello".lower() == "Hello".lower()  # True

# Lexicographic ordering
"apple" < "banana"   # True
"Apple" < "apple"    # True (uppercase < lowercase)
"10" < "9"           # True (string comparison!)
```

---

## 5. Iterating Over Strings

```python
s = "Hello"

# Character by character
for char in s:
    print(char)

# With index
for i, char in enumerate(s):
    print(f"{i}: {char}")

# Reverse iteration
for char in reversed(s):
    print(char)
```

---

## 6. String Type Checks

```python
s = "Hello123"

s.isalpha()     # False (contains digits)
s.isdigit()     # False
s.isalnum()     # True (letters and digits)
s.islower()     # False
s.isupper()     # False
s.isspace()     # False
s.istitle()     # False

"Hello".isalpha()  # True
"12345".isdigit()  # True
"   ".isspace()    # True
"Hello World".istitle()  # True
```

---

## 7. Common Patterns

### Check if String Contains Only Specific Characters
```python
def is_valid_hex(s):
    valid = set("0123456789abcdefABCDEF")
    return all(c in valid for c in s)

# Or using set operations
def is_valid_hex(s):
    return set(s) <= set("0123456789abcdefABCDEF")
```

### Character Frequency
```python
from collections import Counter

s = "hello world"
Counter(s)
# Counter({'l': 3, 'o': 2, ' ': 1, 'h': 1, 'e': 1, 'w': 1, 'r': 1, 'd': 1})
```

### Find All Occurrences
```python
def find_all(s, sub):
    indices = []
    start = 0
    while True:
        i = s.find(sub, start)
        if i == -1:
            break
        indices.append(i)
        start = i + 1
    return indices

find_all("abcabc", "abc")  # [0, 3]
```

---

## Next Steps
- [String Methods](02_string_methods.md)
