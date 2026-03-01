# Reading Files

## 1. Opening Files

```python
# Basic open
f = open("file.txt", "r")
# ... work with file ...
f.close()  # Don't forget!

# Better: context manager
with open("file.txt", "r") as f:
    # ... work with file ...
# Automatically closed
```

### Open Parameters
```python
open(
    file,           # File path
    mode="r",       # Mode (r/w/a/x/b/t/+)
    encoding=None,  # Text encoding (e.g., "utf-8")
    errors=None,    # How to handle encoding errors
    newline=None,   # Newline handling
)
```

### Encoding
```python
# Always specify encoding for text files
with open("file.txt", "r", encoding="utf-8") as f:
    content = f.read()

# Handle encoding errors
with open("file.txt", "r", encoding="utf-8", errors="ignore") as f:
    content = f.read()

# errors options: "strict" (default), "ignore", "replace"
```

---

## 2. Reading Methods

### read() — Read Entire File
```python
with open("file.txt") as f:
    content = f.read()      # Entire file as string
    # or
    chunk = f.read(100)     # First 100 characters

# Note: Loads entire file into memory
```

### readline() — Read One Line
```python
with open("file.txt") as f:
    line1 = f.readline()    # First line (includes \n)
    line2 = f.readline()    # Second line
    # Returns "" when file ends
```

### readlines() — Read All Lines
```python
with open("file.txt") as f:
    lines = f.readlines()   # List of all lines

# Each line includes the newline character
# ["Line 1\n", "Line 2\n", "Line 3\n"]
```

### Iteration — Best for Large Files
```python
with open("file.txt") as f:
    for line in f:
        print(line.strip())  # Process line by line

# Memory efficient: doesn't load entire file
```

---

## 3. Common Patterns

### Read Entire File
```python
# As string
content = Path("file.txt").read_text(encoding="utf-8")

# Or
with open("file.txt", encoding="utf-8") as f:
    content = f.read()
```

### Read Lines (Stripped)
```python
with open("file.txt") as f:
    lines = [line.strip() for line in f]

# Or
with open("file.txt") as f:
    lines = f.read().strip().split("\n")
```

### Process Lines One by One
```python
with open("file.txt") as f:
    for line in f:
        process(line.strip())
```

### Count Lines
```python
with open("file.txt") as f:
    count = sum(1 for _ in f)
```

### Search for Pattern
```python
with open("file.txt") as f:
    for i, line in enumerate(f, 1):
        if "pattern" in line:
            print(f"Found at line {i}: {line.strip()}")
```

### Read CSV (Simple)
```python
with open("data.csv") as f:
    for line in f:
        values = line.strip().split(",")
        print(values)

# Better: use csv module
import csv
with open("data.csv") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

### Read JSON
```python
import json

with open("data.json") as f:
    data = json.load(f)

# Or from string
data = json.loads('{"key": "value"}')
```

---

## 4. Binary Files

```python
# Read binary
with open("image.png", "rb") as f:
    data = f.read()      # Returns bytes

    # Read in chunks
    chunk = f.read(1024)  # 1KB at a time

# Check first bytes (magic numbers)
with open("file", "rb") as f:
    header = f.read(4)
    if header == b"\x89PNG":
        print("It's a PNG!")
```

---

## 5. File Position

```python
with open("file.txt") as f:
    content = f.read()

    # File position is now at end
    print(f.tell())  # Current position

    # Seek back to beginning
    f.seek(0)

    # Read again
    content = f.read()

# seek(offset, whence)
# whence: 0 = start (default), 1 = current, 2 = end
f.seek(0)      # Go to start
f.seek(10)     # Go to byte 10
f.seek(0, 2)   # Go to end
f.seek(-10, 2) # 10 bytes before end (binary mode only)
```

---

## 6. Error Handling

```python
# File not found
try:
    with open("nonexistent.txt") as f:
        content = f.read()
except FileNotFoundError:
    print("File not found")

# Permission denied
except PermissionError:
    print("Permission denied")

# General I/O error
except IOError as e:
    print(f"I/O error: {e}")

# Check before opening
from pathlib import Path

if Path("file.txt").exists():
    with open("file.txt") as f:
        content = f.read()
```

---

## 7. Reading Large Files

```python
# Stream line by line
with open("large_file.txt") as f:
    for line in f:
        process(line)

# Read in chunks
def read_in_chunks(file_path, chunk_size=8192):
    with open(file_path, "rb") as f:
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            yield chunk

for chunk in read_in_chunks("large_file.bin"):
    process(chunk)
```

---

## 8. Practice Problems

1. Count words in a text file.

2. Find all lines containing a specific word.

3. Read a configuration file (key=value format).

4. Compute the MD5 hash of a file.

---

## Next Steps
- [Writing Files](02_writing_files.md)
