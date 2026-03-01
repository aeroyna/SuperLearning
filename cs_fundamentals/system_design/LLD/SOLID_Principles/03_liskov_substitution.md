# Liskov Substitution Principle (LSP)

> "Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."

## Core Concept

If `S` is a subtype of `T`, then objects of type `T` can be replaced with objects of type `S` without altering the correctness of the program.

In simpler terms: **A subclass should behave like its parent class.**

---

## The Classic Example: Rectangle and Square

### The Problem

```python
class Rectangle:
    def __init__(self, width: int, height: int):
        self._width = width
        self._height = height

    @property
    def width(self) -> int:
        return self._width

    @width.setter
    def width(self, value: int):
        self._width = value

    @property
    def height(self) -> int:
        return self._height

    @height.setter
    def height(self, value: int):
        self._height = value

    def area(self) -> int:
        return self._width * self._height


class Square(Rectangle):
    """A square IS-A rectangle mathematically, but..."""

    def __init__(self, side: int):
        super().__init__(side, side)

    @Rectangle.width.setter
    def width(self, value: int):
        self._width = value
        self._height = value  # Must keep square!

    @Rectangle.height.setter
    def height(self, value: int):
        self._width = value
        self._height = value  # Must keep square!
```

### Why It Breaks LSP

```python
def test_rectangle(rect: Rectangle):
    """This test should pass for any Rectangle"""
    rect.width = 5
    rect.height = 4
    assert rect.area() == 20  # 5 * 4 = 20

# Works fine with Rectangle
rectangle = Rectangle(2, 3)
test_rectangle(rectangle)  # ✓ Passes

# Fails with Square!
square = Square(3)
test_rectangle(square)  # ✗ Fails! area is 16 (4*4), not 20
```

### The Solution

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> int:
        pass

class Rectangle(Shape):
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height

    def area(self) -> int:
        return self.width * self.height

class Square(Shape):
    def __init__(self, side: int):
        self.side = side

    def area(self) -> int:
        return self.side * self.side
```

Square and Rectangle are now siblings, not parent-child. Both can substitute for Shape.

---

## Rules for LSP

### 1. Preconditions Cannot Be Strengthened

Subclass methods cannot require more than the base class.

```python
# Bad: Subclass has stricter precondition
class Bird:
    def set_altitude(self, altitude: int):
        """Can set any altitude"""
        self.altitude = altitude

class Penguin(Bird):
    def set_altitude(self, altitude: int):
        if altitude != 0:
            raise ValueError("Penguins can't fly!")  # Stricter!
        self.altitude = altitude

# Good: Separate abstractions
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class FlyingBird(Bird):
    def move(self):
        self.fly()

    def fly(self):
        print("Flying")

class Penguin(Bird):
    def move(self):
        self.swim()

    def swim(self):
        print("Swimming")
```

### 2. Postconditions Cannot Be Weakened

Subclass must deliver at least what the base class promises.

```python
# Bad: Subclass delivers less
class FileReader:
    def read(self, path: str) -> str:
        """Returns file content as string"""
        with open(path) as f:
            return f.read()

class LazyFileReader(FileReader):
    def read(self, path: str) -> str:
        return None  # Violates postcondition!

# Good: Maintain postcondition
class LazyFileReader(FileReader):
    def __init__(self):
        self.cache = {}

    def read(self, path: str) -> str:
        if path not in self.cache:
            self.cache[path] = super().read(path)
        return self.cache[path]  # Still returns string
```

### 3. Invariants Must Be Preserved

Properties that are always true in base class must remain true.

```python
# Bad: Subclass breaks invariant
class BankAccount:
    def __init__(self, balance: float):
        self._balance = balance  # Invariant: balance >= 0

    def withdraw(self, amount: float):
        if amount > self._balance:
            raise ValueError("Insufficient funds")
        self._balance -= amount

class OverdraftAccount(BankAccount):
    def withdraw(self, amount: float):
        self._balance -= amount  # Breaks invariant! Can go negative

# Good: Maintain or explicitly change invariant
class OverdraftAccount(BankAccount):
    def __init__(self, balance: float, overdraft_limit: float):
        super().__init__(balance)
        self.overdraft_limit = overdraft_limit

    def withdraw(self, amount: float):
        if amount > self._balance + self.overdraft_limit:
            raise ValueError("Exceeds overdraft limit")
        self._balance -= amount
```

### 4. History Constraint

Subclass should not remove parent's capabilities.

```python
# Bad: Subclass removes capability
class Switchable:
    def turn_on(self):
        self.is_on = True

    def turn_off(self):
        self.is_on = False

class AlwaysOnLight(Switchable):
    def turn_off(self):
        pass  # Does nothing - breaks expectation!

# Good: Don't inherit if you can't support all operations
class Light:
    def __init__(self):
        self.is_on = False

    def turn_on(self):
        self.is_on = True

    def turn_off(self):
        self.is_on = False

class AlwaysOnLight:
    """Not a Switchable - doesn't inherit"""
    def __init__(self):
        self.is_on = True
```

---

## Real-World Examples

### Example 1: Employee Hierarchy

```python
from abc import ABC, abstractmethod

# Bad Design
class Employee:
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary

    def calculate_bonus(self) -> float:
        return self.salary * 0.1

class Contractor(Employee):
    def calculate_bonus(self) -> float:
        raise NotImplementedError("Contractors don't get bonuses")
        # Violates LSP!

# Good Design
class Worker(ABC):
    def __init__(self, name: str):
        self.name = name

    @abstractmethod
    def calculate_compensation(self) -> float:
        pass

class Employee(Worker):
    def __init__(self, name: str, salary: float):
        super().__init__(name)
        self.salary = salary

    def calculate_compensation(self) -> float:
        return self.salary + self.calculate_bonus()

    def calculate_bonus(self) -> float:
        return self.salary * 0.1

class Contractor(Worker):
    def __init__(self, name: str, hourly_rate: float, hours: int):
        super().__init__(name)
        self.hourly_rate = hourly_rate
        self.hours = hours

    def calculate_compensation(self) -> float:
        return self.hourly_rate * self.hours
```

### Example 2: Collections

```python
from abc import ABC, abstractmethod
from typing import Iterator, Any

class Collection(ABC):
    @abstractmethod
    def add(self, item: Any) -> None:
        pass

    @abstractmethod
    def remove(self, item: Any) -> None:
        pass

    @abstractmethod
    def __iter__(self) -> Iterator:
        pass

class MutableList(Collection):
    def __init__(self):
        self._items = []

    def add(self, item: Any) -> None:
        self._items.append(item)

    def remove(self, item: Any) -> None:
        self._items.remove(item)

    def __iter__(self) -> Iterator:
        return iter(self._items)

# Bad: ImmutableList can't fulfill Collection contract
class ImmutableList(Collection):
    def __init__(self, items: list):
        self._items = tuple(items)

    def add(self, item: Any) -> None:
        raise NotImplementedError("Cannot modify")  # Violates LSP!

    def remove(self, item: Any) -> None:
        raise NotImplementedError("Cannot modify")  # Violates LSP!

# Good: Separate interfaces
class ReadableCollection(ABC):
    @abstractmethod
    def __iter__(self) -> Iterator:
        pass

    @abstractmethod
    def __len__(self) -> int:
        pass

class MutableCollection(ReadableCollection):
    @abstractmethod
    def add(self, item: Any) -> None:
        pass

    @abstractmethod
    def remove(self, item: Any) -> None:
        pass

class ImmutableList(ReadableCollection):
    def __init__(self, items: list):
        self._items = tuple(items)

    def __iter__(self) -> Iterator:
        return iter(self._items)

    def __len__(self) -> int:
        return len(self._items)
```

### Example 3: Database Connections

```python
from abc import ABC, abstractmethod

class DatabaseConnection(ABC):
    @abstractmethod
    def connect(self) -> None:
        pass

    @abstractmethod
    def execute(self, query: str) -> list:
        pass

    @abstractmethod
    def close(self) -> None:
        pass

class MySQLConnection(DatabaseConnection):
    def connect(self) -> None:
        print("Connecting to MySQL")

    def execute(self, query: str) -> list:
        print(f"Executing MySQL query: {query}")
        return []

    def close(self) -> None:
        print("Closing MySQL connection")

class PostgreSQLConnection(DatabaseConnection):
    def connect(self) -> None:
        print("Connecting to PostgreSQL")

    def execute(self, query: str) -> list:
        print(f"Executing PostgreSQL query: {query}")
        return []

    def close(self) -> None:
        print("Closing PostgreSQL connection")

# Both can substitute for DatabaseConnection
def run_migration(db: DatabaseConnection):
    db.connect()
    db.execute("CREATE TABLE users (id INT, name VARCHAR)")
    db.close()

run_migration(MySQLConnection())      # Works
run_migration(PostgreSQLConnection())  # Works
```

---

## LSP Violation Indicators

### Code Smells

1. **Type checking in client code**
   ```python
   if isinstance(obj, SpecificType):
       # Handle differently
   ```

2. **Empty or exception-throwing overrides**
   ```python
   def some_method(self):
       raise NotImplementedError()
   ```

3. **Overriding to do nothing**
   ```python
   def some_method(self):
       pass  # Intentionally empty
   ```

4. **Documentation saying "don't use X with Y"**
   ```python
   """Note: This method doesn't work with subclass Z"""
   ```

---

## How to Design for LSP

### 1. Use Interface Segregation

Create specific interfaces instead of large general ones.

### 2. Prefer Composition Over Inheritance

```python
# Instead of inheriting from a class you can't fully substitute
class Car:
    def __init__(self, engine: Engine):
        self.engine = engine

    def start(self):
        self.engine.start()
```

### 3. Design by Contract

Define clear preconditions, postconditions, and invariants.

### 4. Think About Behavior, Not Just Structure

Ask: "Can this subclass truly do everything the parent can?"

---

## Testing for LSP

```python
import unittest

class ShapeTest(unittest.TestCase):
    """All shape subclasses should pass these tests"""

    def shape_area_is_positive(self, shape):
        self.assertGreater(shape.area(), 0)

    def test_circle(self):
        self.shape_area_is_positive(Circle(5))

    def test_rectangle(self):
        self.shape_area_is_positive(Rectangle(3, 4))

    def test_square(self):
        self.shape_area_is_positive(Square(5))
```

---

## Related Topics

- [[01_single_responsibility|Single Responsibility Principle]]
- [[04_interface_segregation|Interface Segregation Principle]]
- [[../Design_Patterns/00_design_patterns|Design Patterns]]

---

**Tags**: #solid #lsp #design-principles #oop #inheritance
