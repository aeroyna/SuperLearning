# Python Mastery 🐍

Python has evolved from a simple scripting language into one of the most versatile and powerful programming languages in the world. This comprehensive guide covers everything from fundamentals to advanced internals, designed to transform you into a **modern Python developer** who truly understands how Python works under the hood.

> *"Readability counts."* — The Zen of Python

---

## 🗺️ Learning Path

### 1. Basics
Foundation concepts for writing clean, correct Python code.

- [**1.1 Introduction to Python**](Basics/00_introduction_to_python.md) — History, philosophy, installation, and your first program
- [**1.2 Variables and Data Types**](Basics/Variables_and_Data_Types/00_variables_and_data_types.md) — Dynamic typing, built-in types, type system internals
- [**1.3 Operators**](Basics/Operators/00_operators.md) — Arithmetic, comparison, logical, bitwise, and special operators
- [**1.4 Control Flow**](Basics/Control_Flow/00_control_flow.md) — Conditionals, loops, and control statements
- [**1.5 Functions**](Basics/Functions/00_functions.md) — Definition, arguments, scope, and closures
- [**1.6 Built-in Data Structures**](Basics/Data_Structures/00_data_structures.md) — Lists, tuples, dictionaries, and sets
- [**1.7 Strings**](Basics/Strings/00_strings.md) — String manipulation, formatting, and encoding
- [**1.8 File I/O**](Basics/File_IO/00_file_io.md) — Reading, writing, and working with files
- [**1.9 Exception Handling**](Basics/Exception_Handling/00_exception_handling.md) — Try/except, custom exceptions, and best practices

---

### 2. Intermediate
Advanced language features that separate Python beginners from proficient developers.

- [**2.1 Object-Oriented Programming**](Intermediate/OOP/00_oop.md) — Classes, inheritance, polymorphism, and encapsulation
- [**2.2 Functional Programming**](Intermediate/Functional_Programming/00_functional_programming.md) — Lambda, map, filter, reduce, and comprehensions
- [**2.3 Decorators**](Intermediate/Decorators/00_decorators.md) — Function and class decorators, decorator patterns
- [**2.4 Generators and Iterators**](Intermediate/Generators_and_Iterators/00_generators_and_iterators.md) — Yield, iterator protocol, and lazy evaluation
- [**2.5 Context Managers**](Intermediate/Context_Managers/00_context_managers.md) — The `with` statement and resource management
- [**2.6 Modules and Packages**](Intermediate/Modules_and_Packages/00_modules_and_packages.md) — Import system, package structure, and namespaces
- [**2.7 Regular Expressions**](Intermediate/Regular_Expressions/00_regular_expressions.md) — Pattern matching with the `re` module
- [**2.8 Testing**](Intermediate/Testing/00_testing.md) — unittest, pytest, mocking, and TDD

---

### 3. Advanced
Deep-dive into Python's most powerful and complex features.

- [**3.1 Metaclasses**](Advanced/Metaclasses/00_metaclasses.md) — Class creation, `__new__` vs `__init__`, and metaprogramming
- [**3.2 Descriptors**](Advanced/Descriptors/00_descriptors.md) — The descriptor protocol and attribute access
- [**3.3 Concurrency and Multithreading**](Advanced/Concurrency/00_concurrency.md) — Threading, multiprocessing, and the GIL
- [**3.4 Async Programming**](Advanced/Async_Programming/00_async_programming.md) — asyncio, coroutines, and event loops
- [**3.5 Memory Management**](Advanced/Memory_Management/00_memory_management.md) — Reference counting, garbage collection, and memory optimization
- [**3.6 Performance Optimization**](Advanced/Performance/00_performance.md) — Profiling, Cython, and optimization techniques
- [**3.7 Design Patterns**](Advanced/Design_Patterns/00_design_patterns.md) — Pythonic implementations of classic patterns
- [**3.8 CPython Internals**](Advanced/CPython_Internals/00_cpython_internals.md) — Bytecode, PyObject, and interpreter architecture

---

### 4. Modern Python
Contemporary practices for professional Python development.

- [**4.1 Type Hints and Static Typing**](Modern_Python/Type_Hints/00_type_hints.md) — Type annotations, mypy, and typing module
- [**4.2 Virtual Environments**](Modern_Python/Virtual_Environments/00_virtual_environments.md) — venv, virtualenv, and environment management
- [**4.3 Packaging and Distribution**](Modern_Python/Packaging/00_packaging.md) — setuptools, pyproject.toml, and PyPI publishing
- [**4.4 Dependency Management**](Modern_Python/Dependency_Management/00_dependency_management.md) — pip, poetry, and pipenv
- [**4.5 Python 3.10+ Features**](Modern_Python/Modern_Features/00_modern_features.md) — Pattern matching, union types, and latest additions
- [**4.6 Best Practices and Idioms**](Modern_Python/Best_Practices/00_best_practices.md) — PEP 8, Pythonic code, and common pitfalls

---

## 🚀 Quick Reference

### The Zen of Python
```python
import this
```
Key principles: Beautiful > ugly, Explicit > implicit, Simple > complex, Readability counts.

### Essential One-Liners
```python
# List comprehension
squares = [x**2 for x in range(10)]

# Dictionary comprehension
word_lengths = {word: len(word) for word in words}

# Lambda function
add = lambda x, y: x + y

# Ternary operator
result = "yes" if condition else "no"

# Unpacking
first, *middle, last = [1, 2, 3, 4, 5]

# Walrus operator (3.8+)
if (n := len(data)) > 10:
    print(f"List is too long ({n} elements)")
```

### Common Built-in Functions
| Function | Description |
|----------|-------------|
| `len()` | Return length of object |
| `range()` | Generate sequence of numbers |
| `enumerate()` | Return index-value pairs |
| `zip()` | Combine iterables |
| `map()` | Apply function to iterable |
| `filter()` | Filter iterable by predicate |
| `sorted()` | Return sorted list |
| `any()`/`all()` | Boolean checks on iterables |
| `isinstance()` | Check type |
| `getattr()`/`setattr()` | Dynamic attribute access |

---

## 📚 Prerequisites

Before starting this guide, you should have:
- Basic understanding of programming concepts
- A computer with Python 3.10+ installed
- A code editor (VS Code, PyCharm, or similar)

---

## 🎯 Learning Approach

This guide follows a **depth-over-breadth** philosophy:
1. **Understand the "why"** — Not just how to use features, but why they exist
2. **Learn the internals** — How Python implements things under the hood
3. **Practice deliberately** — Each section includes practice problems
4. **Build mental models** — Visualize how data structures and algorithms work in memory

---

## 📖 Topics Index

| Section | Topics |
|---------|--------|
| **Basics** | Variables, Data Types, Operators, Control Flow, Functions, Data Structures, Strings, File I/O, Exceptions |
| **Intermediate** | OOP, Functional Programming, Decorators, Generators, Context Managers, Modules, Regex, Testing |
| **Advanced** | Metaclasses, Descriptors, Concurrency, Async, Memory Management, Performance, Design Patterns, CPython Internals |
| **Modern Python** | Type Hints, Virtual Environments, Packaging, Dependencies, Python 3.10+ Features, Best Practices |
