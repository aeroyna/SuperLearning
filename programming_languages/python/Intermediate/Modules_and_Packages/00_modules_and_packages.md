# Modules and Packages

Understanding Python's module system is essential for organizing code and using third-party libraries.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Modules**](01_modules.md) | Creating and importing modules |
| [**2. Packages**](02_packages.md) | Package structure, __init__.py |
| [**3. Import System**](03_import_system.md) | How imports work, sys.path |

---

## Quick Reference

### Importing
```python
# Import module
import math
math.sqrt(16)

# Import specific items
from math import sqrt, pi
sqrt(16)

# Import with alias
import numpy as np
from collections import defaultdict as dd

# Import all (avoid in production)
from math import *
```

### Module Structure
```python
# mymodule.py

"""Module docstring."""

# Constants
VERSION = "1.0.0"

# Functions
def greet(name):
    return f"Hello, {name}"

# Classes
class MyClass:
    pass

# Script behavior
if __name__ == "__main__":
    print(greet("World"))
```

---

## Package Structure

```
mypackage/
├── __init__.py
├── module1.py
├── module2.py
└── subpackage/
    ├── __init__.py
    └── module3.py
```

### Importing from Packages
```python
# Import module from package
from mypackage import module1
from mypackage.subpackage import module3

# Import item from module
from mypackage.module1 import some_function
```

### __init__.py
```python
# mypackage/__init__.py

# Control what's exported
from .module1 import public_function
from .module2 import PublicClass

__all__ = ['public_function', 'PublicClass']
```

---

## Relative Imports

```python
# Inside a package
from . import sibling_module
from .sibling_module import something
from .. import parent_module
from ..parent_module import something
```

---

## Next Steps
Start with [Modules](01_modules.md).
