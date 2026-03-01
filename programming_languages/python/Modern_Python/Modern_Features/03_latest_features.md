# Latest Python Features (3.11 - 3.12)

Python continues to evolve rapidly. Here are the standout features from recent releases.

## Python 3.11

### 1. Performance Boost
Python 3.11 is 10-60% faster than 3.10 due to the "Faster CPython" project.
*   **Specializing Adaptive Interpreter**: Bytecode adapts to types at runtime.
*   **Faster Function Calls**: Reduced overhead.

### 2. Exception Groups (`except*`)
Handling multiple exceptions at once (useful for AsyncIO or complex tasks).

```python
try:
    raise ExceptionGroup("Nested", [ValueError(1), TypeError(2)])
except* ValueError as e:
    print(f"Caught ValueErrors: {e.exceptions}")
except* TypeError as e:
    print(f"Caught TypeErrors: {e.exceptions}")
```

### 3. `tomllib`
Built-in TOML parsing (reading `pyproject.toml` is now native!).

```python
import tomllib
with open("pyproject.toml", "rb") as f:
    data = tomllib.load(f)
```

## Python 3.12

### 1. F-String Improvements
F-strings are now fully nestable and support quotes inside expressions.

```python
# Valid in 3.12
f"Hello {data['name']}" 
f"{f"{nested}"}"
```

### 2. Per-Interpreter GIL
Python 3.12 lays the groundwork for sub-interpreters to have their own GIL, enabling true multicore parallelism within a single process (via C extensions currently, exposed to Python later).

### 3. Type Parameter Syntax (PEP 695)
Cleaner generic syntax.

```python
# Old
T = TypeVar("T")
def func(x: T) -> T: ...

# New
def func[T](x: T) -> T: ...

class Stack[T]: ...
```

### 4. `override` Decorator
Static enforcement that a method overrides a parent method.

```python
from typing import override

class Parent:
    def foo(self): pass

class Child(Parent):
    @override
    def foo(self): pass # OK
    
    @override
    def bar(self): pass # Type Checker Error (Parent has no bar)
```
