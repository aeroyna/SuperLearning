# Modules

A module is simply a file containing Python definitions and statements. The file name is the module name with the suffix `.py`.

## Importing Modules

```python
import math
print(math.pi)

import math as m
print(m.pi)

from math import pi
print(pi)
```

## `__name__`

When a module is run directly (not imported), `__name__` is set to `"__main__"`. This allows code to execute only when run as a script.

```python
def main():
    print("Running directly")

if __name__ == "__main__":
    main()
```

## Internals
When you import a module:
1.  Python checks `sys.modules` cache.
2.  If not found, it searches `sys.path`.
3.  It compiles it to bytecode (`.pyc`).
4.  It executes the module body to define the names.
