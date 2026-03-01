# Working with Paths

Modern Python uses `pathlib` for path manipulation. For legacy code, `os.path` is still available.

---

## 1. pathlib Basics

```python
from pathlib import Path

# Create path
p = Path("folder/file.txt")
p = Path.home() / "Documents" / "file.txt"

# Current directory
cwd = Path.cwd()
p = Path(".")

# Home directory
home = Path.home()
```

### Path Components
```python
p = Path("/home/user/documents/file.txt")

p.name      # "file.txt"
p.stem      # "file"
p.suffix    # ".txt"
p.suffixes  # [".txt"] (handles .tar.gz: [".tar", ".gz"])
p.parent    # Path("/home/user/documents")
p.parents   # All parents as sequence
p.anchor    # "/" (root on Unix, "C:\\" on Windows)
p.parts     # ("/", "home", "user", "documents", "file.txt")
```

### Path Manipulation
```python
p = Path("/home/user/documents")

# Join paths
p / "subfolder" / "file.txt"
# Path("/home/user/documents/subfolder/file.txt")

# Change parts
p.with_name("report.txt")      # Replace filename
p.with_stem("report")          # Replace stem (Python 3.9+)
p.with_suffix(".md")           # Replace extension

# Resolve to absolute
Path("../file.txt").resolve()  # Full absolute path
```

---

## 2. File Operations

### Check Existence
```python
p = Path("file.txt")

p.exists()      # True if exists
p.is_file()     # True if regular file
p.is_dir()      # True if directory
p.is_symlink()  # True if symbolic link
```

### Read and Write
```python
# Read text
content = Path("file.txt").read_text(encoding="utf-8")

# Write text
Path("file.txt").write_text("Hello!", encoding="utf-8")

# Read bytes
data = Path("image.png").read_bytes()

# Write bytes
Path("image.png").write_bytes(data)
```

### File Info
```python
p = Path("file.txt")

stat = p.stat()
stat.st_size      # Size in bytes
stat.st_mtime     # Modification time (timestamp)
stat.st_ctime     # Creation time (metadata change on Unix)

# Human-readable times
from datetime import datetime
mtime = datetime.fromtimestamp(stat.st_mtime)
```

---

## 3. Directory Operations

### Create Directories
```python
p = Path("new/folder/structure")

p.mkdir()                         # Create (parent must exist)
p.mkdir(parents=True)             # Create all parents
p.mkdir(parents=True, exist_ok=True)  # No error if exists
```

### List Contents
```python
p = Path("folder")

# Immediate children
list(p.iterdir())

# With glob pattern
list(p.glob("*.txt"))         # .txt files in folder
list(p.glob("**/*.txt"))      # .txt files recursively
list(p.rglob("*.txt"))        # Same as **/*.txt

# All files recursively
list(p.rglob("*"))
```

### Delete
```python
# Delete file
Path("file.txt").unlink()
Path("file.txt").unlink(missing_ok=True)  # No error if missing

# Delete empty directory
Path("empty_folder").rmdir()

# Delete directory tree (use shutil)
import shutil
shutil.rmtree("folder")
```

### Rename and Move
```python
p = Path("old_name.txt")

p.rename("new_name.txt")        # Rename
p.rename("other_folder/file.txt")  # Move

# Cross-filesystem move
import shutil
shutil.move("source", "dest")
```

### Copy
```python
import shutil

# Copy file
shutil.copy("source.txt", "dest.txt")
shutil.copy2("source.txt", "dest.txt")  # Preserve metadata

# Copy directory
shutil.copytree("source_dir", "dest_dir")
```

---

## 4. Pattern Matching

```python
p = Path("folder")

# Glob patterns
p.glob("*.txt")          # All .txt files
p.glob("data_*.csv")     # Files matching pattern
p.glob("**/test_*.py")   # Recursive search

# Match specific pattern
Path("file.txt").match("*.txt")    # True
Path("folder/file.txt").match("folder/*.txt")  # True

# Using fnmatch
import fnmatch
fnmatch.fnmatch("file.txt", "*.txt")  # True
fnmatch.filter(filenames, "*.py")      # Filter list
```

---

## 5. Cross-Platform Paths

```python
from pathlib import Path, PurePosixPath, PureWindowsPath

# Automatic for current OS
p = Path("folder/file.txt")

# Force specific style
unix = PurePosixPath("folder/file.txt")
windows = PureWindowsPath("folder/file.txt")

# Convert separators
str(Path("folder/file.txt"))  # Uses OS-appropriate separator
```

---

## 6. os.path (Legacy)

```python
import os.path

# Path operations
os.path.join("folder", "subfolder", "file.txt")
os.path.dirname("/home/user/file.txt")  # "/home/user"
os.path.basename("/home/user/file.txt") # "file.txt"
os.path.splitext("file.txt")            # ("file", ".txt")

# Checks
os.path.exists("file.txt")
os.path.isfile("file.txt")
os.path.isdir("folder")

# Path info
os.path.getsize("file.txt")
os.path.getmtime("file.txt")

# Absolute path
os.path.abspath("relative/path")
os.path.realpath("symlink")  # Resolve symlinks
```

---

## 7. Common Patterns

### Find Files by Extension
```python
def find_files(directory, extension):
    return list(Path(directory).rglob(f"*{extension}"))

python_files = find_files("src", ".py")
```

### Get All Subdirectories
```python
def get_subdirs(directory):
    return [p for p in Path(directory).iterdir() if p.is_dir()]
```

### Create Unique Filename
```python
def unique_path(path):
    p = Path(path)
    counter = 1
    while p.exists():
        p = p.with_stem(f"{p.stem}_{counter}")
        counter += 1
    return p

unique_path("file.txt")  # file_1.txt if file.txt exists
```

### Safe Path Join
```python
from pathlib import Path

def safe_join(base, *paths):
    """Join paths, preventing directory traversal."""
    base = Path(base).resolve()
    target = (base / Path(*paths)).resolve()

    if not str(target).startswith(str(base)):
        raise ValueError("Path traversal detected")

    return target
```

### Walk Directory Tree
```python
# Using os.walk
import os

for root, dirs, files in os.walk("folder"):
    for file in files:
        path = Path(root) / file
        print(path)

# Using pathlib
def walk(path):
    for p in Path(path).iterdir():
        if p.is_dir():
            yield from walk(p)
        else:
            yield p
```

---

## 8. Environment Paths

```python
import os
from pathlib import Path

# Environment variable
Path(os.environ.get("HOME", "/tmp"))
Path(os.environ.get("USERPROFILE", "C:\\Users\\Default"))

# Expanduser (~ to home)
Path("~/.config").expanduser()

# Expand environment variables
os.path.expandvars("$HOME/file.txt")
```

---

## 9. Practice Problems

1. Write a function to find duplicate files by content hash.

2. Create a function to recursively count files by extension.

3. Implement a simple file synchronization tool.

4. Create a function to safely delete old files (> 30 days).
