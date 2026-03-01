# Packages

A package is a way of structuring Python’s module namespace by using "dotted module names". A package is a directory containing a special file `__init__.py`.

## Structure

```
sound/              Top-level package
      __init__.py
      effects/      Subpackage
              __init__.py
              echo.py
      filters/      Subpackage
              __init__.py
              karaoke.py
```

## Importing from Packages

```python
import sound.effects.echo
sound.effects.echo.echofilter(input, output, delay=0.7)

from sound.effects import echo
echo.echofilter(input, output, delay=0.7)
```

## `__init__.py`

Can be empty, or can execute initialization code for the package. It is also used to expose API.

```python
# sound/effects/__init__.py
from .echo import echofilter
# Now users can do: from sound.effects import echofilter
```

## Relative Imports
Inside a package, you can use relative imports.

```python
# Inside sound/effects/echo.py
from .. import filters  # Parent package
from . import reverse   # Sibling module
```
