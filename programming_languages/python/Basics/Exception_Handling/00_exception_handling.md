# Exception Handling

Exception handling allows you to gracefully handle errors and unexpected situations in your code.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Try/Except**](01_try_except.md) | Catching and handling exceptions |
| [**2. Custom Exceptions**](02_custom_exceptions.md) | Creating your own exception types |

---

## Quick Reference

### Basic Try/Except
```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
```

### Multiple Exceptions
```python
try:
    value = int(input())
except ValueError:
    print("Not a valid number")
except KeyboardInterrupt:
    print("User cancelled")
```

### Full Syntax
```python
try:
    # Code that might raise exception
    risky_operation()
except SomeError as e:
    # Handle specific error
    print(f"Error: {e}")
except (Error1, Error2):
    # Handle multiple error types
    pass
except Exception:
    # Catch most exceptions
    pass
else:
    # Runs if no exception
    print("Success!")
finally:
    # Always runs
    cleanup()
```

---

## Common Built-in Exceptions

| Exception | Description |
|-----------|-------------|
| `ValueError` | Invalid value for operation |
| `TypeError` | Wrong type for operation |
| `KeyError` | Dictionary key not found |
| `IndexError` | Sequence index out of range |
| `AttributeError` | Object has no attribute |
| `FileNotFoundError` | File doesn't exist |
| `ZeroDivisionError` | Division by zero |
| `ImportError` | Import failed |
| `NameError` | Name not defined |
| `RuntimeError` | Generic runtime error |

---

## Exception Hierarchy

```
BaseException
├── SystemExit
├── KeyboardInterrupt
├── GeneratorExit
└── Exception
    ├── StopIteration
    ├── ArithmeticError
    │   ├── FloatingPointError
    │   ├── OverflowError
    │   └── ZeroDivisionError
    ├── AttributeError
    ├── LookupError
    │   ├── IndexError
    │   └── KeyError
    ├── OSError
    │   ├── FileNotFoundError
    │   ├── PermissionError
    │   └── ...
    ├── TypeError
    ├── ValueError
    └── ...
```

---

## Best Practices

### Be Specific
```python
# Bad: catches everything
try:
    do_something()
except:
    pass

# Good: catch specific exceptions
try:
    do_something()
except ValueError as e:
    handle_value_error(e)
except TypeError as e:
    handle_type_error(e)
```

### Don't Silence Errors
```python
# Bad: error is hidden
try:
    process(data)
except Exception:
    pass  # Silent failure

# Good: at least log
try:
    process(data)
except Exception as e:
    logger.error(f"Failed to process: {e}")
    raise  # Re-raise if can't handle
```

### Use Exceptions for Exceptional Cases
```python
# Bad: using exceptions for control flow
def find_item(items, target):
    try:
        return items.index(target)
    except ValueError:
        return -1

# Better: check first
def find_item(items, target):
    return items.index(target) if target in items else -1
```

---

## Next Steps
Start with [Try/Except](01_try_except.md).
