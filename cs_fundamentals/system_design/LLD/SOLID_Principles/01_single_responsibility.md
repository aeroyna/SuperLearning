# Single Responsibility Principle (SRP)

> A class should have only one reason to change.

---

## The Problem

```python
# Bad: User class does everything
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def save(self):
        """Save to database"""
        db.execute("INSERT INTO users VALUES (%s, %s)", self.name, self.email)

    def send_welcome_email(self):
        """Send email"""
        smtp.send(self.email, "Welcome!", "Thanks for signing up")

    def generate_report(self):
        """Generate PDF report"""
        pdf = PDF()
        pdf.add_text(f"User: {self.name}")
        return pdf.output()

    def validate(self):
        """Validate user data"""
        if "@" not in self.email:
            raise ValueError("Invalid email")
```

**Problems:**
- Changes to email system affect User class
- Changes to database affect User class
- Changes to reporting affect User class
- Hard to test individual responsibilities
- Class becomes bloated over time

---

## The Solution

```python
# Good: Each class has one responsibility

class User:
    """Domain model - just holds user data"""
    def __init__(self, name, email):
        self.name = name
        self.email = email


class UserValidator:
    """Validates user data"""
    @staticmethod
    def validate(user: User):
        if "@" not in user.email:
            raise ValueError("Invalid email")


class UserRepository:
    """Handles database operations"""
    def __init__(self, db):
        self.db = db

    def save(self, user: User):
        self.db.execute(
            "INSERT INTO users VALUES (%s, %s)",
            user.name, user.email
        )

    def find_by_email(self, email: str) -> User:
        row = self.db.query("SELECT * FROM users WHERE email = %s", email)
        return User(row['name'], row['email'])


class EmailService:
    """Handles email sending"""
    def __init__(self, smtp_client):
        self.smtp = smtp_client

    def send_welcome_email(self, user: User):
        self.smtp.send(
            to=user.email,
            subject="Welcome!",
            body="Thanks for signing up"
        )


class UserReportGenerator:
    """Generates user reports"""
    def generate(self, user: User) -> bytes:
        pdf = PDF()
        pdf.add_text(f"User: {user.name}")
        return pdf.output()
```

---

## Usage

```python
# Using the separated classes
user = User("John", "john@example.com")

# Validate
UserValidator.validate(user)

# Save
repo = UserRepository(db)
repo.save(user)

# Send email
email_service = EmailService(smtp)
email_service.send_welcome_email(user)

# Generate report
report = UserReportGenerator().generate(user)
```

---

## Benefits

### 1. Easy to Test

```python
# Each class can be tested independently
class TestUserValidator(unittest.TestCase):
    def test_valid_email(self):
        user = User("John", "john@example.com")
        UserValidator.validate(user)  # No exception

    def test_invalid_email(self):
        user = User("John", "invalid")
        with self.assertRaises(ValueError):
            UserValidator.validate(user)


class TestUserRepository(unittest.TestCase):
    def test_save(self):
        mock_db = Mock()
        repo = UserRepository(mock_db)
        repo.save(User("John", "john@example.com"))
        mock_db.execute.assert_called_once()
```

### 2. Easy to Modify

```python
# Changing email provider only affects EmailService
class EmailService:
    def __init__(self, email_provider: EmailProvider):
        self.provider = email_provider

    def send_welcome_email(self, user: User):
        self.provider.send(user.email, "Welcome!", "...")

# Switch from SMTP to SendGrid
email_service = EmailService(SendGridProvider())
```

### 3. Easy to Reuse

```python
# UserValidator can be reused anywhere
class RegistrationService:
    def register(self, user: User):
        UserValidator.validate(user)
        # ...

class ProfileUpdateService:
    def update(self, user: User):
        UserValidator.validate(user)
        # ...
```

---

## Identifying SRP Violations

### Signs of Violation

| Sign | Example |
|------|---------|
| "and" in class name | `UserValidatorAndSaver` |
| Many unrelated methods | `save()`, `sendEmail()`, `generateReport()` |
| Different reasons to change | DB schema change AND email template change |
| Hard to name the class | If you can't describe in one sentence |
| Large class file | 500+ lines is a red flag |

### Questions to Ask

1. "What does this class do?" (Should be one sentence)
2. "What could cause this class to change?"
3. "Can I test this without mocking many dependencies?"

---

## Real-World Example: MVC

```
Model-View-Controller separates:

Model: Data and business logic
View: Presentation
Controller: Request handling

Each has ONE responsibility.
```

---

## Common Refactoring

### Extract Class

```python
# Before: One big class
class Order:
    def calculate_total(self): ...
    def apply_discount(self): ...
    def validate(self): ...
    def save(self): ...
    def send_confirmation(self): ...

# After: Multiple focused classes
class Order: ...                    # Domain model
class PriceCalculator: ...          # Pricing logic
class DiscountService: ...          # Discount rules
class OrderValidator: ...           # Validation
class OrderRepository: ...          # Persistence
class OrderNotificationService: ... # Notifications
```

---

## Interview Tips

1. **Spot violations** in given code
2. **Propose refactoring** with clear separation
3. **Name the pattern**: "I'm applying Single Responsibility Principle by..."
4. **Show the benefit**: "This makes it easier to test/modify/extend"
