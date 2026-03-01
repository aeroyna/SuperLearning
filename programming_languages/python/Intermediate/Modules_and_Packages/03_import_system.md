# The Import System

Python's import system is complex and practically exposed via the `importlib` module.

## `sys.modules`
A dictionary that maps module names to modules which have already been loaded. This is the first place Python checks.

```python
import sys
print(sys.modules.keys())
```

## `sys.path`
A list of strings specifying the search path for modules.
1.  Directory containing the input script (or current directory).
2.  PYTHONPATH (a list of directory names, with same syntax as shell variable PATH).
3.  The installation-dependent default (site-packages).

## Finders and Loaders (Meta-path)
`sys.meta_path` contains a list of finder objects.
1.  **Finder**: `find_spec()` returns a `ModuleSpec`.
2.  **Loader**: `exec_module()` executes the module.

You can create custom importers (e.g., to import from S3 or a zip file) by adding to `sys.meta_path`.

```python
import importlib.abc
import sys

class MyFinder(importlib.abc.MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        if fullname == 'my_virtual_module':
            # Return spec...
            pass
        return None

sys.meta_path.insert(0, MyFinder())
```
