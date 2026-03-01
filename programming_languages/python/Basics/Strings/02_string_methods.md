# String Methods

All string methods return new strings (strings are immutable).

---

## 1. Case Conversion

```python
s = "Hello, World!"

s.upper()       # "HELLO, WORLD!"
s.lower()       # "hello, world!"
s.capitalize()  # "Hello, world!"
s.title()       # "Hello, World!"
s.swapcase()    # "hELLO, wORLD!"

# Case-insensitive comparison
"Hello".casefold() == "hello".casefold()  # True
# casefold is more aggressive than lower() for Unicode
"ß".casefold()  # "ss" (German sharp S)
```

---

## 2. Searching

```python
s = "Hello, World!"

# Find position (returns -1 if not found)
s.find("World")      # 7
s.find("Python")     # -1
s.find("o")          # 4 (first occurrence)
s.rfind("o")         # 8 (last occurrence)

# Index (raises ValueError if not found)
s.index("World")     # 7
# s.index("Python")  # ValueError

# Count occurrences
s.count("o")         # 2

# Startswith / endswith
s.startswith("Hello")  # True
s.endswith("!")        # True
s.startswith(("Hi", "Hello"))  # True (tuple of prefixes)
```

---

## 3. Stripping Whitespace

```python
s = "   Hello, World!   \n"

s.strip()       # "Hello, World!"
s.lstrip()      # "Hello, World!   \n"
s.rstrip()      # "   Hello, World!"

# Strip specific characters
"###Hello###".strip("#")   # "Hello"
"www.example.com".lstrip("w.")  # "example.com"
```

---

## 4. Splitting and Joining

### Split
```python
s = "Hello, World, Python"

s.split()              # ['Hello,', 'World,', 'Python'] (by whitespace)
s.split(", ")          # ['Hello', 'World', 'Python']
s.split(",")           # ['Hello', ' World', ' Python']

# Limit splits
"a,b,c,d".split(",", 2)  # ['a', 'b', 'c,d']

# Split from right
"a,b,c,d".rsplit(",", 2)  # ['a,b', 'c', 'd']

# Split lines
"line1\nline2\nline3".splitlines()  # ['line1', 'line2', 'line3']
```

### Join
```python
", ".join(["a", "b", "c"])   # "a, b, c"
"".join(["a", "b", "c"])     # "abc"
"\n".join(["line1", "line2"])  # "line1\nline2"

# Must be strings
# ", ".join([1, 2, 3])  # TypeError
", ".join(map(str, [1, 2, 3]))  # "1, 2, 3"
```

### Partition
```python
s = "name=value"

s.partition("=")   # ('name', '=', 'value')
s.rpartition("=")  # ('name', '=', 'value')

# If separator not found
"hello".partition("=")  # ('hello', '', '')
```

---

## 5. Replacing

```python
s = "Hello, World!"

s.replace("World", "Python")   # "Hello, Python!"
s.replace("o", "0")            # "Hell0, W0rld!"
s.replace("o", "0", 1)         # "Hell0, World!" (limit replacements)
```

---

## 6. Alignment and Padding

```python
s = "Hello"

# Center
s.center(11)       # "   Hello   "
s.center(11, "-")  # "---Hello---"

# Left/Right justify
s.ljust(10)        # "Hello     "
s.rjust(10)        # "     Hello"
s.rjust(10, "0")   # "00000Hello"

# Zero padding
"42".zfill(5)      # "00042"
"-42".zfill(5)     # "-0042"
```

---

## 7. Character Type Checks

```python
"Hello".isalpha()    # True (all letters)
"12345".isdigit()    # True (all digits)
"Hello123".isalnum() # True (letters and digits)
"   ".isspace()      # True (all whitespace)
"hello".islower()    # True
"HELLO".isupper()    # True
"Hello World".istitle()  # True (title case)
"print".isidentifier()   # True (valid Python identifier)
"True".isidentifier()    # True
"3var".isidentifier()    # False

# Numeric checks
"123".isnumeric()    # True
"½".isnumeric()      # True (includes fractions)
"²".isnumeric()      # True (includes superscripts)
"一二三".isnumeric()  # True (Chinese numerals)
```

---

## 8. Encoding and Decoding

```python
# String to bytes
s = "Hello, 世界!"
b = s.encode("utf-8")
# b'Hello, \xe4\xb8\x96\xe7\x95\x8c!'

# Bytes to string
b.decode("utf-8")  # "Hello, 世界!"

# Common encodings
s.encode("ascii", errors="ignore")   # b'Hello, !'
s.encode("ascii", errors="replace")  # b'Hello, ???!'
```

---

## 9. Translation Tables

```python
# Create translation table
table = str.maketrans("aeiou", "12345")
"hello world".translate(table)  # "h2ll4 w4rld"

# Delete characters
table = str.maketrans("", "", "aeiou")
"hello world".translate(table)  # "hll wrld"

# Multiple operations
table = str.maketrans({"a": "4", "e": "3", "o": "0"})
"hello".translate(table)  # "h3ll0"
```

---

## 10. Expanding Tabs

```python
"Hello\tWorld".expandtabs(4)  # "Hello   World"
"01\t012\t0123".expandtabs(4)  # "01  012 0123"
```

---

## 11. Common Patterns

### Remove Punctuation
```python
import string

s = "Hello, World! How are you?"
s.translate(str.maketrans("", "", string.punctuation))
# "Hello World How are you"
```

### Word Count
```python
def word_count(s):
    return len(s.split())

word_count("Hello World")  # 2
```

### Title Case with Exceptions
```python
def smart_title(s, exceptions={"a", "an", "the", "of"}):
    words = s.lower().split()
    result = [words[0].capitalize()]
    result.extend(
        word if word in exceptions else word.capitalize()
        for word in words[1:]
    )
    return " ".join(result)

smart_title("the lord of the rings")
# "The Lord of the Rings"
```

### Check Palindrome
```python
def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]

is_palindrome("A man a plan a canal Panama")  # True
```

---

## 12. Method Chaining

```python
s = "  Hello, World!  "

result = s.strip().lower().replace(",", "").split()
# ['hello', 'world!']

# Multi-line for readability
result = (s
    .strip()
    .lower()
    .replace(",", "")
    .split())
```

---

## Next Steps
- [String Formatting](03_string_formatting.md)
