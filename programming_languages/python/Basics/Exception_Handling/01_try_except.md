# Try/Except

## 1. Basic Syntax

```python
try:
    # Code that might raise an exception
    result = risky_operation()
except SomeException:
    # Handle the exception
    print("An error occurred")
```

### Catching Specific Exceptions
```python
try:
    x = int("not a number")
except ValueError:
    print("Invalid number format")
```

### Accessing the Exception Object
```python
try:
    x = 1 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")       # "Error: division by zero"
    print(f"Type: {type(e)}")  # "Type: <class 'ZeroDivisionError'>"
```

---

## 2. Multiple Exceptions

### Separate Handlers
```python
try:
    value = my_dict[key]
    result = 10 / value
except KeyError:
    print("Key not found")
except ZeroDivisionError:
    print("Cannot divide by zero")
```

### Combined Handler
```python
try:
    do_something()
except (ValueError, TypeError) as e:
    print(f"Value or type error: {e}")
```

### Catch All (Use Sparingly)
```python
try:
    risky_operation()
except Exception as e:
    print(f"Something went wrong: {e}")
    # Log and re-raise if you can't handle it
    raise
```

---

## 3. else Clause

Runs if **no exception** was raised:

```python
try:
    result = calculate()
except CalculationError:
    print("Calculation failed")
else:
    # Only runs if no exception
    save_result(result)
    print("Success!")
```

### Why Use else?
```python
# Without else — unclear what we're protecting
try:
    value = get_value()
    processed = process(value)  # Is this protected?
    save(processed)             # Is this protected?
except ValueError:
    handle_error()

# With else — clear separation
try:
    value = get_value()  # Only this is protected
except ValueError:
    handle_error()
else:
    processed = process(value)  # These run if no exception
    save(processed)
```

---

## 4. finally Clause

**Always** runs, even if exception is raised or return is called:

```python
try:
    file = open("data.txt")
    process(file.read())
except FileNotFoundError:
    print("File not found")
finally:
    file.close()  # Always runs

# Better: use context manager
with open("data.txt") as file:
    process(file.read())
```

### Use Cases
```python
# Cleanup resources
def process_data():
    connection = connect_to_database()
    try:
        result = connection.execute(query)
        return result
    finally:
        connection.close()  # Always closes

# Restore state
original_value = settings.DEBUG
try:
    settings.DEBUG = True
    run_tests()
finally:
    settings.DEBUG = original_value
```

---

## 5. Raising Exceptions

### Basic Raise
```python
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

### Re-raising
```python
try:
    risky_operation()
except SomeError:
    log_error()
    raise  # Re-raise the same exception
```

### Exception Chaining
```python
try:
    parse_config()
except FileNotFoundError as e:
    raise ConfigError("Config file missing") from e

# Shows: ConfigError ... caused by FileNotFoundError
```

### Suppress Original Exception
```python
try:
    do_something()
except SomeError:
    raise DifferentError("New error") from None
    # Original exception not shown
```

---

## 6. Exception Groups (Python 3.11+)

Handle multiple exceptions that occur simultaneously:

```python
def process_items(items):
    errors = []
    for item in items:
        try:
            process(item)
        except Exception as e:
            errors.append(e)

    if errors:
        raise ExceptionGroup("Processing failed", errors)

try:
    process_items(items)
except* ValueError as eg:
    print(f"Value errors: {eg.exceptions}")
except* TypeError as eg:
    print(f"Type errors: {eg.exceptions}")
```

---

## 7. Common Patterns

### EAFP (Easier to Ask Forgiveness than Permission)
```python
# Pythonic
try:
    value = my_dict[key]
except KeyError:
    value = default

# Or even simpler
value = my_dict.get(key, default)
```

### LBYL (Look Before You Leap)
```python
# Less Pythonic but sometimes clearer
if key in my_dict:
    value = my_dict[key]
else:
    value = default
```

### Guard Clauses with Exceptions
```python
def process_user(user_id):
    user = get_user(user_id)
    if user is None:
        raise ValueError(f"User {user_id} not found")

    if not user.is_active:
        raise PermissionError("User is not active")

    # Main logic here
    return process(user)
```

### Contextual Exception Handling
```python
class FileProcessor:
    def process(self, path):
        try:
            return self._process_file(path)
        except FileNotFoundError:
            raise ProcessingError(f"Input file not found: {path}")
        except PermissionError:
            raise ProcessingError(f"Cannot read file: {path}")
        except Exception as e:
            raise ProcessingError(f"Failed to process {path}") from e
```

### Retry Pattern
```python
import time

def retry(func, max_attempts=3, delay=1):
    for attempt in range(max_attempts):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts - 1:
                raise
            print(f"Attempt {attempt + 1} failed: {e}")
            time.sleep(delay)

result = retry(unstable_operation, max_attempts=5, delay=2)
```

---

## 8. Exception Context

```python
import sys
import traceback

try:
    1 / 0
except ZeroDivisionError:
    # Current exception info
    exc_type, exc_value, exc_tb = sys.exc_info()

    # Print traceback
    traceback.print_exc()

    # Get traceback as string
    tb_str = traceback.format_exc()
```

---

## 9. Assertions

For debugging and development:

```python
def calculate_percentage(value, total):
    assert total != 0, "Total cannot be zero"
    assert 0 <= value <= total, "Value out of range"
    return (value / total) * 100

# Assertions can be disabled with -O flag
# python -O script.py
```

---

## 10. Practice Problems

1. Write a function that safely converts a string to an integer, returning a default value on failure.

2. Implement a retry decorator that retries a function on specified exceptions.

3. Create a context manager that suppresses specified exceptions.

4. Write a function that validates user input with appropriate exceptions.

---

## Next Steps
- [Custom Exceptions](02_custom_exceptions.md)
