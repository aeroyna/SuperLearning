# Classes and Objects

## 1. Defining a Class

```python
class Dog:
    """A simple Dog class."""

    # Class attribute (shared by all instances)
    species = "Canis familiaris"

    # Initializer (constructor)
    def __init__(self, name, age):
        # Instance attributes (unique to each instance)
        self.name = name
        self.age = age

    # Instance method
    def bark(self):
        return f"{self.name} says Woof!"

    def description(self):
        return f"{self.name} is {self.age} years old"
```

---

## 2. Creating Objects (Instantiation)

```python
# Create instances
buddy = Dog("Buddy", 3)
max = Dog("Max", 5)

# Access attributes
print(buddy.name)      # "Buddy"
print(buddy.species)   # "Canis familiaris"

# Call methods
print(buddy.bark())    # "Buddy says Woof!"
print(max.description())  # "Max is 5 years old"
```

---

## 3. The `self` Parameter

`self` refers to the instance calling the method:

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self  # Allows method chaining

    def get_count(self):
        return self.count

c = Counter()
c.increment().increment().increment()
print(c.get_count())  # 3
```

---

## 4. `__init__` Method

The initializer sets up the object's initial state:

```python
class Person:
    def __init__(self, name, age=0):
        self.name = name
        self.age = age
        self._created_at = datetime.now()  # Internal state

    def __repr__(self):
        return f"Person({self.name!r}, {self.age})"

# Different ways to create
p1 = Person("Alice", 30)
p2 = Person("Bob")  # Uses default age=0
```

### `__init__` vs `__new__`
```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self.value = 42

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

---

## 5. Instance Attributes

```python
class Person:
    def __init__(self, name):
        self.name = name          # Public attribute
        self._age = 0             # "Protected" (convention)
        self.__secret = "hidden"  # "Private" (name mangling)

p = Person("Alice")
p.name        # "Alice"
p._age        # 0 (accessible, but indicates internal use)
# p.__secret  # AttributeError
p._Person__secret  # "hidden" (name-mangled name)
```

### Dynamic Attributes
```python
p = Person("Alice")
p.email = "alice@example.com"  # Add attribute dynamically
print(p.email)  # "alice@example.com"

# Check attribute existence
hasattr(p, 'email')  # True
getattr(p, 'phone', 'N/A')  # 'N/A' (default)

# Set attribute dynamically
setattr(p, 'phone', '555-1234')
```

---

## 6. Class Attributes

```python
class Employee:
    # Class attribute
    company = "TechCorp"
    employee_count = 0

    def __init__(self, name):
        self.name = name
        Employee.employee_count += 1

e1 = Employee("Alice")
e2 = Employee("Bob")

print(Employee.employee_count)  # 2
print(e1.company)               # "TechCorp"

# Modifying class attribute
Employee.company = "NewCorp"
print(e1.company)  # "NewCorp"
print(e2.company)  # "NewCorp"

# Creating instance attribute shadows class attribute
e1.company = "OtherCorp"
print(e1.company)  # "OtherCorp"
print(e2.company)  # "NewCorp"
```

---

## 7. Methods

### Instance Methods
```python
class Calculator:
    def add(self, a, b):
        return a + b

calc = Calculator()
calc.add(3, 5)  # 8
```

### Class Methods
```python
class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def from_string(cls, date_string):
        year, month, day = map(int, date_string.split('-'))
        return cls(year, month, day)

    @classmethod
    def today(cls):
        from datetime import date
        d = date.today()
        return cls(d.year, d.month, d.day)

d1 = Date(2024, 1, 15)
d2 = Date.from_string("2024-01-15")
d3 = Date.today()
```

### Static Methods
```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b

    @staticmethod
    def is_even(n):
        return n % 2 == 0

Math.add(3, 5)      # 8
Math.is_even(4)     # True
```

---

## 8. The `__slots__` Attribute

Optimize memory for many instances:

```python
class Point:
    __slots__ = ('x', 'y')

    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(1, 2)
# p.z = 3  # AttributeError: 'Point' object has no attribute 'z'

# Memory comparison
import sys
class PointNoSlots:
    def __init__(self, x, y):
        self.x = x
        self.y = y

sys.getsizeof(PointNoSlots(1, 2).__dict__)  # ~100 bytes
# Point has no __dict__, uses less memory
```

---

## 9. Documentation

```python
class BankAccount:
    """
    A bank account with deposit and withdrawal capabilities.

    Attributes:
        owner: The name of the account owner.
        balance: The current balance in dollars.

    Example:
        >>> account = BankAccount("Alice", 100)
        >>> account.deposit(50)
        >>> account.balance
        150
    """

    def __init__(self, owner, balance=0):
        """Initialize account with owner name and optional balance."""
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        """Add amount to balance. Amount must be positive."""
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount
```

---

## 10. Practice Problems

1. Create a `Book` class with title, author, and pages, with a method to display book info.

2. Create a `Rectangle` class with width and height, and methods for area and perimeter.

3. Create a `BankAccount` class with deposit, withdraw, and transfer methods.

4. Implement a `Stack` class using a list internally.

---

## Next Steps
- [Inheritance](02_inheritance.md)
