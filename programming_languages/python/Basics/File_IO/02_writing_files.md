# Writing Files

## 1. Writing Modes

| Mode | Description |
|------|-------------|
| `"w"` | Write (truncates existing) |
| `"a"` | Append (keeps existing) |
| `"x"` | Exclusive create (fails if exists) |
| `"w+"` | Write and read |

---

## 2. Writing Methods

### write() — Write String
```python
with open("file.txt", "w") as f:
    f.write("Hello, World!")
    f.write("\n")  # Newline not automatic
    f.write("Second line\n")
```

### writelines() — Write Multiple Strings
```python
lines = ["Line 1\n", "Line 2\n", "Line 3\n"]

with open("file.txt", "w") as f:
    f.writelines(lines)

# Note: writelines doesn't add newlines automatically
```

### print() — With file Parameter
```python
with open("file.txt", "w") as f:
    print("Hello, World!", file=f)
    print("Line 2", file=f)  # Adds newline automatically
    print(1, 2, 3, sep=", ", file=f)  # "1, 2, 3"
```

---

## 3. Common Patterns

### Write Text File
```python
# Using open
with open("file.txt", "w", encoding="utf-8") as f:
    f.write("Hello, World!\n")
    f.write("Second line\n")

# Using pathlib
from pathlib import Path
Path("file.txt").write_text("Hello, World!", encoding="utf-8")
```

### Write Lines
```python
lines = ["Line 1", "Line 2", "Line 3"]

# Add newlines
with open("file.txt", "w") as f:
    for line in lines:
        f.write(line + "\n")

# Or using join
with open("file.txt", "w") as f:
    f.write("\n".join(lines) + "\n")

# Or using print
with open("file.txt", "w") as f:
    for line in lines:
        print(line, file=f)
```

### Append to File
```python
with open("log.txt", "a") as f:
    f.write(f"[{datetime.now()}] New entry\n")
```

### Safe Overwrite (Check First)
```python
from pathlib import Path

path = Path("file.txt")
if path.exists():
    response = input(f"{path} exists. Overwrite? (y/n): ")
    if response.lower() != "y":
        print("Cancelled")
        exit()

path.write_text("New content")
```

### Exclusive Create
```python
try:
    with open("file.txt", "x") as f:
        f.write("New file content")
except FileExistsError:
    print("File already exists!")
```

---

## 4. Binary Files

```python
# Write binary
data = b"\x89PNG\r\n\x1a\n..."

with open("image.png", "wb") as f:
    f.write(data)

# Copy binary file
with open("source.bin", "rb") as src:
    with open("dest.bin", "wb") as dst:
        dst.write(src.read())

# Copy in chunks (memory efficient)
with open("source.bin", "rb") as src:
    with open("dest.bin", "wb") as dst:
        while chunk := src.read(8192):
            dst.write(chunk)
```

---

## 5. Writing Structured Data

### CSV
```python
import csv

data = [
    ["Name", "Age", "City"],
    ["Alice", 30, "NYC"],
    ["Bob", 25, "LA"],
]

with open("data.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerows(data)
```

### JSON
```python
import json

data = {"name": "Alice", "age": 30, "cities": ["NYC", "LA"]}

with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Pretty print
json.dump(data, f, indent=2, sort_keys=True)
```

### INI/Config
```python
import configparser

config = configparser.ConfigParser()
config["DEFAULT"] = {"server": "localhost", "port": "8080"}
config["database"] = {"host": "db.example.com", "port": "5432"}

with open("config.ini", "w") as f:
    config.write(f)
```

---

## 6. Atomic Writes

Write to temp file, then rename (prevents corruption):

```python
import tempfile
import os
from pathlib import Path

def atomic_write(path, content):
    path = Path(path)

    # Write to temp file in same directory
    with tempfile.NamedTemporaryFile(
        mode="w",
        dir=path.parent,
        delete=False
    ) as tmp:
        tmp.write(content)
        tmp_path = tmp.name

    # Atomic rename
    os.replace(tmp_path, path)

atomic_write("file.txt", "Safe content")
```

---

## 7. Buffering

```python
# Default: fully buffered for files
with open("file.txt", "w") as f:
    f.write("data")  # May not be written immediately

# Force write
with open("file.txt", "w") as f:
    f.write("data")
    f.flush()  # Force write to disk

# Line buffered (useful for logs)
with open("log.txt", "w", buffering=1) as f:
    f.write("Line 1\n")  # Written immediately after newline

# Unbuffered (binary only)
with open("file.bin", "wb", buffering=0) as f:
    f.write(b"data")  # Written immediately
```

---

## 8. Temporary Files

```python
import tempfile

# Temporary file (auto-deleted)
with tempfile.NamedTemporaryFile(mode="w", delete=True) as f:
    f.write("Temporary content")
    f.flush()
    # File exists here
# File deleted after with block

# Keep temporary file
with tempfile.NamedTemporaryFile(mode="w", delete=False) as f:
    f.write("Persistent temp")
    temp_path = f.name

# Temporary directory
with tempfile.TemporaryDirectory() as tmpdir:
    temp_file = Path(tmpdir) / "file.txt"
    temp_file.write_text("Temp content")
# Directory and contents deleted
```

---

## 9. Error Handling

```python
from pathlib import Path

try:
    with open("file.txt", "w") as f:
        f.write("Content")
except PermissionError:
    print("Permission denied")
except IOError as e:
    print(f"I/O error: {e}")

# Create parent directories
path = Path("new/folder/file.txt")
path.parent.mkdir(parents=True, exist_ok=True)
path.write_text("Content")
```

---

## 10. Practice Problems

1. Create a simple logging function that appends timestamps.

2. Write a function that saves a dictionary to JSON atomically.

3. Create a CSV file from a list of dictionaries.

4. Implement a file backup function before overwriting.

---

## Next Steps
- [Working with Paths](03_paths.md)
