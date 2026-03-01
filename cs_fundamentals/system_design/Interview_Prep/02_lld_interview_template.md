# LLD Interview Template

Use this template as a mental checklist during Low Level Design (OOD) interviews.

---

## Phase 1: Requirements Clarification (5 minutes)

### Questions to Ask

```markdown
**Core Functionality**
- What are the main use cases?
- What entities/objects are involved?
- What actions can each entity perform?

**Constraints & Assumptions**
- Is this single-threaded or multi-threaded?
- Do we need persistence?
- Any performance constraints?
- Scale: How many concurrent users/objects?

**Scope**
- What should I focus on?
- Should I implement any specific patterns?
- Console-based or API-based interface?
```

### Output
```
Use Cases:
1. [Actor] can [action]
2. [Actor] can [action]
3. System should [capability]

Entities:
- Entity1: [brief description]
- Entity2: [brief description]

Constraints:
- Multi-threaded: Yes/No
- Persistence: Yes/No
- Scale: [approximate numbers]

Focus Area: [specific component]
```

---

## Phase 2: Identify Core Objects (5 minutes)

### Object Identification Techniques

```markdown
**Noun Analysis**
- Extract nouns from requirements → candidate classes
- Filter out attributes (properties of objects)
- Keep entities that have behavior or state

**Verb Analysis**
- Extract verbs → candidate methods
- Map verbs to the classes that perform them
```

### Output
```
Core Classes:
┌─────────────────────────────────────────────────────────────┐
│  Class Name     │  Responsibility                          │
├─────────────────────────────────────────────────────────────┤
│  [Class1]       │  [single responsibility]                 │
│  [Class2]       │  [single responsibility]                 │
│  [Class3]       │  [single responsibility]                 │
└─────────────────────────────────────────────────────────────┘

Key Relationships:
- Class1 HAS-A Class2 (composition)
- Class3 IS-A Class4 (inheritance)
- Class5 USES Class6 (dependency)
```

---

## Phase 3: Define Interfaces & Abstract Classes (5 minutes)

### Apply SOLID Principles

```markdown
**S - Single Responsibility**
- Each class does ONE thing well
- If you need "and" to describe it, split it

**O - Open/Closed**
- Use interfaces for extension points
- New features = new classes, not modifications

**L - Liskov Substitution**
- Subtypes must be substitutable for base types
- Don't override methods with incompatible behavior

**I - Interface Segregation**
- Many specific interfaces > one general interface
- Clients shouldn't depend on unused methods

**D - Dependency Inversion**
- Depend on abstractions, not concretions
- Inject dependencies via constructor
```

### Output
```python
# Define interfaces for key behaviors
class IMovable(ABC):
    @abstractmethod
    def move(self, direction: Direction) -> bool:
        pass

class IPaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: Decimal) -> PaymentResult:
        pass

# Use enums for fixed sets of values
class Status(Enum):
    PENDING = "pending"
    ACTIVE = "active"
    COMPLETED = "completed"
```

---

## Phase 4: Design Core Classes (15 minutes)

### Class Design Template

```python
class ClassName:
    """
    Single responsibility description.

    Relationships:
    - Owns: [composed objects]
    - Uses: [dependencies]
    - Implements: [interfaces]
    """

    def __init__(self, dependency: IDependency):
        # Private state
        self._state: StateType = initial_value
        self._items: List[Item] = []

        # Injected dependencies
        self._dependency = dependency

    # Public interface
    def public_action(self, param: Type) -> ReturnType:
        """Document the behavior."""
        self._validate(param)
        result = self._internal_logic(param)
        self._notify_observers(result)
        return result

    # Private helpers
    def _validate(self, param: Type) -> None:
        if not self._is_valid(param):
            raise ValidationError("...")

    # Properties for controlled access
    @property
    def state(self) -> StateType:
        return self._state
```

### Class Diagram (ASCII)

```
┌─────────────────────────────────┐
│         <<interface>>           │
│           IVehicle              │
├─────────────────────────────────┤
│ + start(): void                 │
│ + stop(): void                  │
│ + get_position(): Position      │
└─────────────────────────────────┘
              △
              │ implements
    ┌─────────┴─────────┐
    │                   │
┌───────────────┐  ┌───────────────┐
│     Car       │  │  Motorcycle   │
├───────────────┤  ├───────────────┤
│ - engine      │  │ - engine      │
│ - wheels: 4   │  │ - wheels: 2   │
├───────────────┤  ├───────────────┤
│ + start()     │  │ + start()     │
│ + stop()      │  │ + stop()      │
└───────────────┘  └───────────────┘
        │ has-a
        ▼
┌───────────────┐
│    Engine     │
├───────────────┤
│ - horsepower  │
│ - fuel_type   │
├───────────────┤
│ + ignite()    │
│ + shutdown()  │
└───────────────┘
```

---

## Phase 5: Apply Design Patterns (5-10 minutes)

### Pattern Selection Guide

| Problem | Pattern | When to Use |
|---------|---------|-------------|
| Create objects without specifying class | **Factory** | Multiple types with shared interface |
| Complex object construction | **Builder** | Many optional parameters |
| One instance globally | **Singleton** | Config, connection pool, cache |
| Add behavior dynamically | **Decorator** | Extend without inheritance |
| Simplify complex subsystem | **Facade** | Multiple classes, one interface |
| Define algorithm skeleton | **Template Method** | Same steps, different details |
| Switch behavior at runtime | **Strategy** | Multiple algorithms, one interface |
| React to state changes | **Observer** | Notify multiple objects of changes |
| Handle state transitions | **State** | Object behavior depends on state |
| Chain handlers | **Chain of Responsibility** | Multiple handlers, unknown which applies |

### Pattern Implementation Example

```python
# Strategy Pattern Example
class PricingStrategy(ABC):
    @abstractmethod
    def calculate_price(self, base_price: Decimal) -> Decimal:
        pass

class RegularPricing(PricingStrategy):
    def calculate_price(self, base_price: Decimal) -> Decimal:
        return base_price

class WeekendPricing(PricingStrategy):
    def calculate_price(self, base_price: Decimal) -> Decimal:
        return base_price * Decimal("1.5")

class Booking:
    def __init__(self, pricing: PricingStrategy):
        self._pricing = pricing

    def get_total(self, base_price: Decimal) -> Decimal:
        return self._pricing.calculate_price(base_price)
```

---

## Phase 6: Handle Edge Cases & Concurrency (5 minutes)

### Concurrency Considerations

```python
import threading
from typing import Optional

class ThreadSafeCache:
    """Thread-safe implementation using locks."""

    _instance: Optional['ThreadSafeCache'] = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                # Double-checked locking
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        self._cache: Dict[str, Any] = {}
        self._cache_lock = threading.RLock()

    def get(self, key: str) -> Optional[Any]:
        with self._cache_lock:
            return self._cache.get(key)

    def put(self, key: str, value: Any) -> None:
        with self._cache_lock:
            self._cache[key] = value
```

### Error Handling

```python
# Custom exceptions for domain errors
class DomainException(Exception):
    """Base exception for domain errors."""
    pass

class InsufficientFundsError(DomainException):
    def __init__(self, available: Decimal, required: Decimal):
        self.available = available
        self.required = required
        super().__init__(f"Need {required}, have {available}")

class ResourceNotFoundError(DomainException):
    def __init__(self, resource_type: str, resource_id: str):
        super().__init__(f"{resource_type} not found: {resource_id}")
```

---

## Phase 7: Wrap Up (5 minutes)

### Walk Through a Use Case

```markdown
**Trace a complete flow:**

1. User calls `system.action(params)`
2. System validates input via `Validator`
3. System finds/creates `Entity` via `Repository`
4. Entity executes business logic
5. Observer notifies dependent components
6. System returns result to user

**Example for Parking Lot:**
1. Vehicle arrives at `Entrance`
2. `ParkingLot.findSpot(vehicle)` called
3. `SpotAllocationStrategy` finds suitable spot
4. `ParkingTicket` created with entry time
5. Vehicle parks in assigned `ParkingSpot`
6. Display boards updated via Observer
```

### Extensibility Discussion

```markdown
**How to add new features:**

1. "Add motorcycle parking"
   → Create `MotorcycleSpot extends ParkingSpot`
   → Add to `SpotAllocationStrategy`
   → No changes to existing code (Open/Closed)

2. "Add valet parking"
   → Create `ValetService` class
   → Implement `IParkingService` interface
   → Inject into `ParkingLot`

3. "Add different pricing"
   → Create new `PricingStrategy` implementation
   → Configure via dependency injection
```

---

## Checklist

```markdown
[ ] Clarified requirements and scope
[ ] Identified core entities/objects
[ ] Defined interfaces for key behaviors
[ ] Applied appropriate design patterns
[ ] Followed SOLID principles
[ ] Handled edge cases and errors
[ ] Considered thread safety (if applicable)
[ ] Drew class diagram showing relationships
[ ] Walked through at least one use case
[ ] Discussed extensibility
```

---

## Common LLD Interview Problems

| Problem | Key Patterns | Key Classes |
|---------|--------------|-------------|
| Parking Lot | Strategy, Factory | ParkingLot, Spot, Vehicle, Ticket |
| Elevator | State, Strategy, Observer | Elevator, Floor, Request, Scheduler |
| LRU Cache | - | Node, DoublyLinkedList, HashMap |
| Chess | State, Strategy, Factory | Board, Piece, Player, Move |
| Booking System | Strategy, Observer | Show, Seat, Booking, Payment |
| Vending Machine | State | VendingMachine, Product, Coin, State |
| File System | Composite | File, Directory, FileSystem |
| Snake Game | - | Snake, Board, Food, Direction |

---

## Quick Reference: UML Notation

```
┌─────────────────────────────────────────────────────────────────┐
│  Relationship          │  Notation   │  Meaning                │
├─────────────────────────────────────────────────────────────────┤
│  Inheritance           │  ───△       │  IS-A                   │
│  Implementation        │  ---△       │  Implements interface   │
│  Composition           │  ───◆       │  Owns, lifecycle bound  │
│  Aggregation           │  ───◇       │  Has-A, independent     │
│  Dependency            │  --->       │  Uses                   │
│  Association           │  ───        │  Knows about            │
└─────────────────────────────────────────────────────────────────┘

Visibility:
+ public
- private
# protected
~ package
```
