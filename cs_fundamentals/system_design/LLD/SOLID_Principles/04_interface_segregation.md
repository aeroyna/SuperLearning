# Interface Segregation Principle (ISP)

> "No client should be forced to depend on methods it does not use."

## Core Concept

- Prefer many small, specific interfaces over one large, general interface
- Clients should only know about methods that are relevant to them
- Split "fat" interfaces into smaller, cohesive ones

---

## The Problem: Fat Interfaces

```python
from abc import ABC, abstractmethod

# Bad: One interface for all workers
class Worker(ABC):
    @abstractmethod
    def work(self) -> None:
        pass

    @abstractmethod
    def eat(self) -> None:
        pass

    @abstractmethod
    def sleep(self) -> None:
        pass

    @abstractmethod
    def attend_meeting(self) -> None:
        pass

    @abstractmethod
    def take_vacation(self) -> None:
        pass

class Human(Worker):
    def work(self) -> None:
        print("Working")

    def eat(self) -> None:
        print("Eating")

    def sleep(self) -> None:
        print("Sleeping")

    def attend_meeting(self) -> None:
        print("In meeting")

    def take_vacation(self) -> None:
        print("On vacation")

class Robot(Worker):
    def work(self) -> None:
        print("Working")

    def eat(self) -> None:
        pass  # Robots don't eat!

    def sleep(self) -> None:
        pass  # Robots don't sleep!

    def attend_meeting(self) -> None:
        pass  # Robots don't attend meetings!

    def take_vacation(self) -> None:
        pass  # Robots don't take vacation!
```

**Problems:**
- `Robot` must implement methods that don't make sense
- Empty implementations indicate interface is too broad
- Changes to interface affect unrelated classes

---

## The Solution: Segregated Interfaces

```python
from abc import ABC, abstractmethod

# Segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self) -> None:
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self) -> None:
        pass

class Sleepable(ABC):
    @abstractmethod
    def sleep(self) -> None:
        pass

class Meetable(ABC):
    @abstractmethod
    def attend_meeting(self) -> None:
        pass

class Vacationable(ABC):
    @abstractmethod
    def take_vacation(self) -> None:
        pass

# Human implements all relevant interfaces
class Human(Workable, Eatable, Sleepable, Meetable, Vacationable):
    def work(self) -> None:
        print("Working")

    def eat(self) -> None:
        print("Eating")

    def sleep(self) -> None:
        print("Sleeping")

    def attend_meeting(self) -> None:
        print("In meeting")

    def take_vacation(self) -> None:
        print("On vacation")

# Robot only implements what it needs
class Robot(Workable):
    def work(self) -> None:
        print("Working efficiently")
```

---

## Real-World Examples

### Example 1: Document Processing

```python
# Bad: Fat interface
class Document(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

    @abstractmethod
    def write(self, content: str) -> None:
        pass

    @abstractmethod
    def delete(self) -> None:
        pass

    @abstractmethod
    def print_doc(self) -> None:
        pass

    @abstractmethod
    def fax(self) -> None:
        pass

    @abstractmethod
    def scan(self) -> None:
        pass

# Good: Segregated interfaces
class Readable(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

class Writable(ABC):
    @abstractmethod
    def write(self, content: str) -> None:
        pass

class Deletable(ABC):
    @abstractmethod
    def delete(self) -> None:
        pass

class Printable(ABC):
    @abstractmethod
    def print_doc(self) -> None:
        pass

class Faxable(ABC):
    @abstractmethod
    def fax(self) -> None:
        pass

class Scannable(ABC):
    @abstractmethod
    def scan(self) -> None:
        pass

# Specific documents implement only what they need
class TextDocument(Readable, Writable, Deletable, Printable):
    def __init__(self, content: str = ""):
        self.content = content

    def read(self) -> str:
        return self.content

    def write(self, content: str) -> None:
        self.content = content

    def delete(self) -> None:
        self.content = ""

    def print_doc(self) -> None:
        print(self.content)

class ReadOnlyDocument(Readable):
    def __init__(self, content: str):
        self.content = content

    def read(self) -> str:
        return self.content
```

### Example 2: User Permissions

```python
# Bad: One interface for all actions
class UserActions(ABC):
    @abstractmethod
    def view_content(self) -> None: pass

    @abstractmethod
    def create_content(self) -> None: pass

    @abstractmethod
    def edit_content(self) -> None: pass

    @abstractmethod
    def delete_content(self) -> None: pass

    @abstractmethod
    def manage_users(self) -> None: pass

    @abstractmethod
    def view_analytics(self) -> None: pass

# Good: Role-based interfaces
class Viewer(ABC):
    @abstractmethod
    def view_content(self) -> None:
        pass

class Creator(ABC):
    @abstractmethod
    def create_content(self) -> None:
        pass

class Editor(ABC):
    @abstractmethod
    def edit_content(self) -> None:
        pass

class Deleter(ABC):
    @abstractmethod
    def delete_content(self) -> None:
        pass

class UserManager(ABC):
    @abstractmethod
    def manage_users(self) -> None:
        pass

class AnalyticsViewer(ABC):
    @abstractmethod
    def view_analytics(self) -> None:
        pass

# Different user types
class GuestUser(Viewer):
    def view_content(self) -> None:
        print("Viewing content")

class RegularUser(Viewer, Creator):
    def view_content(self) -> None:
        print("Viewing content")

    def create_content(self) -> None:
        print("Creating content")

class Moderator(Viewer, Creator, Editor, Deleter):
    def view_content(self) -> None:
        print("Viewing content")

    def create_content(self) -> None:
        print("Creating content")

    def edit_content(self) -> None:
        print("Editing content")

    def delete_content(self) -> None:
        print("Deleting content")

class Admin(Viewer, Creator, Editor, Deleter, UserManager, AnalyticsViewer):
    def view_content(self) -> None:
        print("Viewing content")

    def create_content(self) -> None:
        print("Creating content")

    def edit_content(self) -> None:
        print("Editing content")

    def delete_content(self) -> None:
        print("Deleting content")

    def manage_users(self) -> None:
        print("Managing users")

    def view_analytics(self) -> None:
        print("Viewing analytics")
```

### Example 3: Vehicle Features

```python
from abc import ABC, abstractmethod

# Segregated vehicle capability interfaces
class Drivable(ABC):
    @abstractmethod
    def drive(self) -> None:
        pass

    @abstractmethod
    def brake(self) -> None:
        pass

class Flyable(ABC):
    @abstractmethod
    def take_off(self) -> None:
        pass

    @abstractmethod
    def land(self) -> None:
        pass

class Sailable(ABC):
    @abstractmethod
    def sail(self) -> None:
        pass

    @abstractmethod
    def anchor(self) -> None:
        pass

class Refuelable(ABC):
    @abstractmethod
    def refuel(self, amount: float) -> None:
        pass

class Rechargeable(ABC):
    @abstractmethod
    def recharge(self) -> None:
        pass

# Specific vehicles
class Car(Drivable, Refuelable):
    def drive(self) -> None:
        print("Driving")

    def brake(self) -> None:
        print("Braking")

    def refuel(self, amount: float) -> None:
        print(f"Refueling {amount} liters")

class ElectricCar(Drivable, Rechargeable):
    def drive(self) -> None:
        print("Driving silently")

    def brake(self) -> None:
        print("Regenerative braking")

    def recharge(self) -> None:
        print("Charging battery")

class Airplane(Flyable, Refuelable):
    def take_off(self) -> None:
        print("Taking off")

    def land(self) -> None:
        print("Landing")

    def refuel(self, amount: float) -> None:
        print(f"Adding {amount} gallons of jet fuel")

class FlyingCar(Drivable, Flyable, Refuelable):
    def drive(self) -> None:
        print("Driving on road")

    def brake(self) -> None:
        print("Braking")

    def take_off(self) -> None:
        print("Converting to flight mode")

    def land(self) -> None:
        print("Converting to drive mode")

    def refuel(self, amount: float) -> None:
        print(f"Refueling {amount} liters")
```

---

## Interface Composition

### Combining Interfaces

```python
# Combine related interfaces into composite interfaces
class BasicVehicle(Drivable, Refuelable):
    """Standard vehicle capabilities"""
    pass

class AmphibiousVehicle(Drivable, Sailable, Refuelable):
    """Can operate on land and water"""
    pass

# Use composite interface
class Truck(BasicVehicle):
    def drive(self) -> None:
        print("Driving truck")

    def brake(self) -> None:
        print("Air brakes engaged")

    def refuel(self, amount: float) -> None:
        print(f"Refueling {amount} liters diesel")
```

---

## Benefits of ISP

| Benefit | Description |
|---------|-------------|
| **Decoupling** | Classes only depend on what they use |
| **Flexibility** | Easy to add new implementations |
| **Testability** | Mock only relevant interfaces |
| **Maintainability** | Changes don't ripple unnecessarily |
| **Clarity** | Interfaces express clear purpose |

---

## Signs of ISP Violation

### 1. Empty Method Implementations
```python
def method(self):
    pass  # Not applicable
```

### 2. Raise NotImplementedError
```python
def method(self):
    raise NotImplementedError("Not supported")
```

### 3. Return Null/None
```python
def method(self):
    return None  # Doesn't apply
```

### 4. Large Interfaces
```python
class DoEverything(ABC):
    # 20+ abstract methods...
```

---

## ISP in Practice

### Python's typing Protocol

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

class Movable(Protocol):
    def move(self, x: int, y: int) -> None: ...

class Resizable(Protocol):
    def resize(self, scale: float) -> None: ...

# Any class with these methods works
class Circle:
    def draw(self) -> None:
        print("Drawing circle")

    def move(self, x: int, y: int) -> None:
        print(f"Moving to ({x}, {y})")

    def resize(self, scale: float) -> None:
        print(f"Scaling by {scale}")

# Duck typing - no explicit inheritance needed
def render(obj: Drawable) -> None:
    obj.draw()

render(Circle())  # Works!
```

### Dependency Injection with ISP

```python
class EmailService:
    def __init__(self, sender: Sendable, logger: Loggable):
        self.sender = sender
        self.logger = logger

    def send_email(self, to: str, subject: str, body: str):
        self.logger.log(f"Sending email to {to}")
        self.sender.send(to, subject, body)
```

---

## Related Topics

- [[01_single_responsibility|Single Responsibility Principle]]
- [[05_dependency_inversion|Dependency Inversion Principle]]
- [[../Design_Patterns/Structural/01_adapter|Adapter Pattern]]

---

**Tags**: #solid #isp #design-principles #oop #interfaces
