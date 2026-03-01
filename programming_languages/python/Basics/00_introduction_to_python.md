# Introduction to Python

## 1. History and Philosophy

### The Birth of Python
Python was created by **Guido van Rossum** and first released in **1991**. The name comes from "Monty Python's Flying Circus," reflecting Guido's sense of humor. Python was designed to be a successor to the ABC language, addressing its limitations while keeping its best features.

### Key Historical Milestones
| Year | Version | Significance |
|------|---------|--------------|
| 1991 | 0.9.0 | First public release |
| 2000 | 2.0 | List comprehensions, garbage collection |
| 2008 | 3.0 | Major redesign (not backward-compatible) |
| 2020 | 3.9 | Dictionary merge operators |
| 2021 | 3.10 | Structural pattern matching |
| 2022 | 3.11 | 10-60% faster than 3.10 |
| 2023 | 3.12 | Per-interpreter GIL, improved errors |

### The Zen of Python (PEP 20)
```python
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

These principles guide Python's design and should guide your code.

---

## 2. Why Python?

### Strengths
1. **Readability**: Clean syntax enforces readable code
2. **Versatility**: Web, data science, AI, scripting, automation
3. **Rich Ecosystem**: 400,000+ packages on PyPI
4. **Community**: One of the largest programming communities
5. **Cross-Platform**: Runs on Windows, macOS, Linux, and more

### Use Cases
| Domain | Popular Libraries/Frameworks |
|--------|------------------------------|
| Web Development | Django, Flask, FastAPI |
| Data Science | pandas, NumPy, SciPy |
| Machine Learning | TensorFlow, PyTorch, scikit-learn |
| Automation | Selenium, Beautiful Soup, requests |
| DevOps | Ansible, Fabric, boto3 |
| Scientific Computing | matplotlib, SymPy, Jupyter |

### Compared to Other Languages
```
Python:    print("Hello, World!")
Java:      System.out.println("Hello, World!");
C++:       std::cout << "Hello, World!" << std::endl;
JavaScript: console.log("Hello, World!");
```

Python's philosophy: **"There should be one—and preferably only one—obvious way to do it."**

---

## 3. Python Implementations

Python is a **specification**, and there are multiple implementations:

| Implementation | Language | Use Case |
|---------------|----------|----------|
| **CPython** | C | Reference implementation, most common |
| **PyPy** | Python/RPython | JIT compilation, faster execution |
| **Jython** | Java | JVM integration |
| **IronPython** | C# | .NET integration |
| **MicroPython** | C | Embedded systems, microcontrollers |

### CPython Architecture (High-Level)
```
Source Code (.py)
       ↓
    Lexer/Parser
       ↓
  Abstract Syntax Tree (AST)
       ↓
    Compiler
       ↓
   Bytecode (.pyc)
       ↓
  Python Virtual Machine (PVM)
       ↓
    Execution
```

---

## 4. Installation and Setup

### Installing Python

**Windows:**
1. Download from [python.org](https://python.org)
2. Run installer, **check "Add Python to PATH"**
3. Verify: `python --version`

**macOS:**
```bash
# Using Homebrew (recommended)
brew install python

# Verify
python3 --version
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
python3 --version
```

### Version Manager (Recommended for Development)
Use **pyenv** to manage multiple Python versions:

```bash
# Install pyenv
curl https://pyenv.run | bash

# Install specific version
pyenv install 3.12.0

# Set global version
pyenv global 3.12.0

# Set local version (per-project)
pyenv local 3.11.0
```

### Virtual Environments
Always use virtual environments for projects:

```bash
# Create virtual environment
python -m venv .venv

# Activate (Linux/macOS)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Deactivate
deactivate
```

---

## 5. Your First Python Program

### Interactive Mode (REPL)
```python
$ python3
Python 3.12.0 (main, Oct  2 2023, 00:00:00) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello, World!")
Hello, World!
>>> 2 + 2
4
>>> exit()
```

### Script Mode
Create a file `hello.py`:
```python
#!/usr/bin/env python3
"""My first Python program."""

def main():
    name = input("What is your name? ")
    print(f"Hello, {name}!")

if __name__ == "__main__":
    main()
```

Run it:
```bash
python hello.py
```

### Understanding `if __name__ == "__main__":`
This idiom allows a file to work both as a script and as an importable module:

```python
# greetings.py
def say_hello(name):
    return f"Hello, {name}!"

if __name__ == "__main__":
    # This only runs when executed directly
    print(say_hello("World"))
```

```python
# another_file.py
from greetings import say_hello  # The main block doesn't run

print(say_hello("Python"))
```

---

## 6. Python Development Tools

### IDEs and Editors
| Tool | Strengths |
|------|-----------|
| **VS Code** | Free, extensible, great Python extension |
| **PyCharm** | Full-featured IDE, excellent debugging |
| **Jupyter** | Interactive notebooks for data science |
| **Vim/Neovim** | Terminal-based, highly customizable |

### Essential CLI Tools
```bash
# Package manager
pip install package_name

# Code formatter
pip install black
black my_code.py

# Linter
pip install flake8
flake8 my_code.py

# Type checker
pip install mypy
mypy my_code.py

# Interactive shell
pip install ipython
ipython
```

### Project Structure (Best Practice)
```
my_project/
├── pyproject.toml      # Project metadata and dependencies
├── README.md
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_main.py
└── .venv/              # Virtual environment (not in git)
```

---

## 7. Python 2 vs Python 3

Python 2 reached **end of life on January 1, 2020**. Always use Python 3.

Key differences to be aware of (when reading legacy code):

| Feature | Python 2 | Python 3 |
|---------|----------|----------|
| Print | `print "hello"` | `print("hello")` |
| Division | `5/2 = 2` | `5/2 = 2.5` |
| Unicode | ASCII by default | Unicode by default |
| range() | Returns list | Returns iterator |
| input() | `raw_input()` | `input()` |

---

## 8. Practice Problems

### Problem 1: Hello, You!
Write a program that asks for the user's name and age, then prints a personalized greeting.

**Expected output:**
```
What is your name? Alice
What is your age? 25
Hello, Alice! You will be 26 next year.
```

### Problem 2: Simple Calculator
Create a program that asks for two numbers and an operator (+, -, *, /), then prints the result.

### Problem 3: Temperature Converter
Write a program that converts Celsius to Fahrenheit and vice versa.

Formula: `F = C × 9/5 + 32`

---

## 9. Key Takeaways

1. **Python's philosophy** emphasizes readability and simplicity
2. **CPython** is the reference implementation you'll use
3. **Always use virtual environments** for projects
4. **Python 3** is the only supported version—never start new projects in Python 2
5. The **REPL** is great for experimentation and learning

---

## Next Steps
- [Variables and Data Types](Variables_and_Data_Types/00_variables_and_data_types.md)
