# Custom Exceptions

Creating custom exceptions makes your code more expressive and error handling more precise.

---

## 1. Basic Custom Exception

```python
class MyError(Exception):
    """Base exception for my application."""
    pass

raise MyError("Something went wrong")
```

### With Default Message
```python
class ConfigurationError(Exception):
    """Raised when configuration is invalid."""

    def __init__(self, message="Configuration error"):
        self.message = message
        super().__init__(self.message)

raise ConfigurationError()  # Uses default
raise ConfigurationError("Missing API key")  # Custom message
```

---

## 2. Exception with Additional Data

```python
class ValidationError(Exception):
    """Raised when validation fails."""

    def __init__(self, message, field=None, value=None):
        self.message = message
        self.field = field
        self.value = value
        super().__init__(self.message)

    def __str__(self):
        if self.field:
            return f"{self.field}: {self.message}"
        return self.message

# Usage
raise ValidationError("must be positive", field="age", value=-5)

try:
    validate_user(data)
except ValidationError as e:
    print(f"Field: {e.field}")
    print(f"Value: {e.value}")
    print(f"Error: {e.message}")
```

---

## 3. Exception Hierarchy

```python
# Base exception for your application
class AppError(Exception):
    """Base exception for application errors."""
    pass

# Specific exceptions inherit from base
class DatabaseError(AppError):
    """Database-related errors."""
    pass

class ConnectionError(DatabaseError):
    """Database connection errors."""
    pass

class QueryError(DatabaseError):
    """SQL query errors."""
    pass

class ValidationError(AppError):
    """Validation errors."""
    pass

class AuthError(AppError):
    """Authentication/Authorization errors."""
    pass

# Usage
try:
    process_data()
except DatabaseError:
    # Catches ConnectionError and QueryError too
    print("Database problem")
except AppError:
    # Catches all app-specific errors
    print("Application error")
```

---

## 4. Rich Exception Classes

```python
class HTTPError(Exception):
    """HTTP-related errors with status code."""

    def __init__(self, status_code, message=None, response=None):
        self.status_code = status_code
        self.message = message or self._default_message()
        self.response = response
        super().__init__(self.message)

    def _default_message(self):
        messages = {
            400: "Bad Request",
            401: "Unauthorized",
            403: "Forbidden",
            404: "Not Found",
            500: "Internal Server Error",
        }
        return messages.get(self.status_code, "Unknown Error")

    def __str__(self):
        return f"HTTP {self.status_code}: {self.message}"

    def __repr__(self):
        return f"HTTPError({self.status_code}, {self.message!r})"

# Usage
raise HTTPError(404)  # HTTP 404: Not Found
raise HTTPError(400, "Invalid user ID")  # HTTP 400: Invalid user ID
```

---

## 5. Multiple Error Details

```python
class MultipleValidationErrors(Exception):
    """Container for multiple validation errors."""

    def __init__(self, errors=None):
        self.errors = errors or []
        message = self._build_message()
        super().__init__(message)

    def _build_message(self):
        return "; ".join(str(e) for e in self.errors)

    def add(self, error):
        self.errors.append(error)

    def __bool__(self):
        return bool(self.errors)

    def __iter__(self):
        return iter(self.errors)

# Usage
errors = MultipleValidationErrors()

if not data.get("name"):
    errors.add(ValidationError("required", "name"))
if not data.get("email"):
    errors.add(ValidationError("required", "email"))

if errors:
    raise errors
```

---

## 6. Context-Aware Exceptions

```python
class FileProcessingError(Exception):
    """Error during file processing with context."""

    def __init__(self, message, filename=None, line_number=None):
        self.message = message
        self.filename = filename
        self.line_number = line_number
        super().__init__(self._full_message())

    def _full_message(self):
        parts = [self.message]
        if self.filename:
            parts.append(f"in {self.filename}")
        if self.line_number:
            parts.append(f"at line {self.line_number}")
        return " ".join(parts)

# Usage
raise FileProcessingError(
    "Invalid JSON",
    filename="config.json",
    line_number=42
)
# "Invalid JSON in config.json at line 42"
```

---

## 7. Exception Factory

```python
class ErrorCode:
    """Error codes with factory methods."""

    USER_NOT_FOUND = "E001"
    INVALID_INPUT = "E002"
    PERMISSION_DENIED = "E003"

class AppException(Exception):
    """Application exception with error codes."""

    def __init__(self, code, message, **kwargs):
        self.code = code
        self.message = message
        self.details = kwargs
        super().__init__(f"[{code}] {message}")

    @classmethod
    def user_not_found(cls, user_id):
        return cls(
            ErrorCode.USER_NOT_FOUND,
            f"User {user_id} not found",
            user_id=user_id
        )

    @classmethod
    def invalid_input(cls, field, reason):
        return cls(
            ErrorCode.INVALID_INPUT,
            f"Invalid {field}: {reason}",
            field=field,
            reason=reason
        )

# Usage
raise AppException.user_not_found(123)
raise AppException.invalid_input("email", "must be valid email")
```

---

## 8. Retryable Exceptions

```python
class RetryableError(Exception):
    """Exception that indicates operation can be retried."""

    def __init__(self, message, retry_after=None):
        self.message = message
        self.retry_after = retry_after  # Seconds to wait
        super().__init__(message)

class PermanentError(Exception):
    """Exception that indicates operation should not be retried."""
    pass

# Usage in retry logic
def with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except RetryableError as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(e.retry_after or 1)
        except PermanentError:
            raise  # Don't retry
```

---

## 9. Best Practices

### Inherit from Exception, Not BaseException
```python
# Good
class MyError(Exception):
    pass

# Bad — catches KeyboardInterrupt
class MyError(BaseException):
    pass
```

### Document Your Exceptions
```python
class ConfigError(Exception):
    """Raised when configuration is invalid.

    Attributes:
        key: The configuration key that caused the error.
        message: Explanation of why the configuration is invalid.

    Example:
        >>> raise ConfigError("port", "must be between 1-65535")
    """

    def __init__(self, key, message):
        self.key = key
        self.message = message
        super().__init__(f"Config error for '{key}': {message}")
```

### Keep Exception Hierarchy Shallow
```python
# Good: 2-3 levels deep
AppError
├── NetworkError
├── DatabaseError
└── ValidationError

# Bad: too deep
AppError
└── DataError
    └── ValidationError
        └── FieldValidationError
            └── EmailValidationError
```

### Use Exceptions for Exceptional Cases
```python
# Bad: using exception for control flow
def find_user(user_id):
    try:
        return users[user_id]
    except KeyError:
        return None

# Good: return None or use Optional
def find_user(user_id):
    return users.get(user_id)
```

---

## 10. Practice Problems

1. Create an exception hierarchy for a banking application (InsufficientFunds, AccountLocked, InvalidTransaction).

2. Build a ValidationError that can hold multiple field errors.

3. Create a RateLimitError that includes retry-after time.

4. Implement an exception that can serialize to JSON for API responses.
