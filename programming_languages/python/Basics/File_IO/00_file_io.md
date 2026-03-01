# File I/O

Working with files is fundamental to most Python programs. This section covers reading, writing, and managing files.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Reading Files**](01_reading_files.md) | Open, read, and process file content |
| [**2. Writing Files**](02_writing_files.md) | Create and write to files |
| [**3. Working with Paths**](03_paths.md) | pathlib and os.path |

---

## Quick Reference

### Reading
```python
# Read entire file
with open("file.txt", "r") as f:
    content = f.read()

# Read lines
with open("file.txt", "r") as f:
    for line in f:
        print(line.strip())
```

### Writing
```python
# Write (overwrites)
with open("file.txt", "w") as f:
    f.write("Hello, World!")

# Append
with open("file.txt", "a") as f:
    f.write("New line\n")
```

### Binary Files
```python
# Read binary
with open("image.png", "rb") as f:
    data = f.read()

# Write binary
with open("image.png", "wb") as f:
    f.write(data)
```

---

## File Modes

| Mode | Description |
|------|-------------|
| `"r"` | Read (default) |
| `"w"` | Write (truncates) |
| `"a"` | Append |
| `"x"` | Exclusive create (fails if exists) |
| `"b"` | Binary mode |
| `"t"` | Text mode (default) |
| `"+"` | Read and write |

Common combinations:
- `"r"` — Read text
- `"w"` — Write text (overwrites)
- `"a"` — Append text
- `"rb"` — Read binary
- `"wb"` — Write binary
- `"r+"` — Read and write

---

## The `with` Statement

Always use `with` to ensure proper file handling:

```python
# Good: automatically closes file
with open("file.txt") as f:
    content = f.read()
# File is closed here, even if exception occurs

# Bad: may leave file open on error
f = open("file.txt")
content = f.read()
f.close()  # May not execute if error above
```

---

## pathlib (Modern Path Handling)

```python
from pathlib import Path

# Create path
p = Path("folder") / "subfolder" / "file.txt"

# Read/write shortcuts
content = Path("file.txt").read_text()
Path("file.txt").write_text("Hello!")

# Path operations
p.exists()      # Check existence
p.is_file()     # Is it a file?
p.is_dir()      # Is it a directory?
p.parent        # Parent directory
p.name          # Filename with extension
p.stem          # Filename without extension
p.suffix        # File extension
```

---

## Next Steps
Start with [Reading Files](01_reading_files.md).
