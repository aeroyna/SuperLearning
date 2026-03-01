# Low Level Design (LLD)

Low Level Design focuses on object-oriented design, design patterns, and implementing specific components. LLD interviews assess your ability to write clean, maintainable, extensible code.

---

## What is LLD?

LLD involves:
- **Class design**: Objects, relationships, responsibilities
- **Design patterns**: Reusable solutions to common problems
- **SOLID principles**: Guidelines for good OOP design
- **UML diagrams**: Visualizing class structure

---

## LLD vs HLD

| Aspect | HLD | LLD |
|--------|-----|-----|
| Focus | System architecture | Class/code structure |
| Scale | Millions of users | Single machine |
| Components | Services, databases | Classes, interfaces |
| Diagrams | Architecture diagrams | Class diagrams, sequence diagrams |
| Output | Component design | Pseudo-code or actual code |

---

## LLD Interview Structure

```
┌────────────────────────────────────────────────────────────────┐
│  1. Requirements (5 min)                                       │
│     - Clarify use cases                                        │
│     - Identify actors                                          │
│     - Define scope                                             │
├────────────────────────────────────────────────────────────────┤
│  2. Use Cases (5 min)                                          │
│     - List key use cases                                       │
│     - Prioritize for discussion                                │
├────────────────────────────────────────────────────────────────┤
│  3. Class Design (15-20 min)                                   │
│     - Identify classes                                         │
│     - Define relationships                                     │
│     - Apply design patterns                                    │
├────────────────────────────────────────────────────────────────┤
│  4. Implementation (15-20 min)                                 │
│     - Key methods                                              │
│     - Edge cases                                               │
│     - Concurrency (if applicable)                              │
├────────────────────────────────────────────────────────────────┤
│  5. Extensions (5 min)                                         │
│     - How would you add feature X?                             │
│     - Trade-offs                                               │
└────────────────────────────────────────────────────────────────┘
```

---

## Topics in This Section

### [10. SOLID Principles](SOLID_Principles/00_solid_principles.md)
Foundation of good object-oriented design.
- Single Responsibility
- Open/Closed
- Liskov Substitution
- Interface Segregation
- Dependency Inversion

### [11. Design Patterns](Design_Patterns/00_design_patterns.md)
Reusable solutions to common problems.
- Creational: Singleton, Factory, Builder
- Structural: Adapter, Decorator, Facade
- Behavioral: Strategy, Observer, Command

### [12. LLD Case Studies](Case_Studies/00_case_studies.md)
Practice problems with detailed solutions.
- Parking Lot
- Elevator System
- LRU Cache
- Chess Game

### [13. UML Diagrams](UML_Diagrams/00_uml_diagrams.md)
Visualizing designs.
- Class Diagrams
- Sequence Diagrams
- Use Case Diagrams

---

## LLD Approach

### 1. Clarify Requirements
```
"Design a parking lot"

Questions:
- How many levels/floors?
- Vehicle types (car, motorcycle, truck)?
- Pricing model?
- Entry/exit system?
- Reservations?
```

### 2. Identify Core Objects
```
Nouns in requirements → Classes

Parking Lot: ParkingLot, Level, ParkingSpot
Vehicles: Vehicle, Car, Motorcycle, Truck
Operations: Ticket, Payment
```

### 3. Define Relationships
```
Composition: ParkingLot has Levels has ParkingSpots
Inheritance: Car extends Vehicle
Association: Ticket references Vehicle and ParkingSpot
```

### 4. Apply SOLID Principles
```
Single Responsibility: Separate ParkingSpot from Payment
Open/Closed: Add new vehicle types without modifying existing code
Dependency Inversion: Depend on Vehicle interface, not concrete Car
```

### 5. Choose Design Patterns
```
Strategy: Different pricing strategies
Factory: Create different vehicle types
Singleton: ParkingLot instance
Observer: Notify when spot becomes available
```

---

## Key Skills for LLD

### 1. Identify Classes and Responsibilities
```
Each class should have ONE reason to change.

Bad:
class ParkingLot {
    parkVehicle()
    calculateFee()
    printReceipt()
    sendEmail()
}

Good:
class ParkingLot { parkVehicle(), removeVehicle() }
class FeeCalculator { calculate() }
class ReceiptPrinter { print() }
class NotificationService { send() }
```

### 2. Design for Extension
```
New requirement: Add electric vehicle with charging

Bad: Modify Vehicle class
Good: Add ElectricVehicle subclass, ChargingSpot subclass
```

### 3. Use Interfaces
```
interface Parkable {
    getSize()
    getType()
}

class Car implements Parkable { ... }
class Motorcycle implements Parkable { ... }

// Code depends on Parkable, not Car/Motorcycle
```

---

## Common LLD Problems

### Frequently Asked
1. **Parking Lot** - Core OOP concepts
2. **Elevator System** - State machine, scheduling
3. **LRU Cache** - Data structures
4. **Chess/Tic-Tac-Toe** - Game logic
5. **Library Management** - CRUD operations

### Moderately Asked
6. **Vending Machine** - State pattern
7. **ATM** - Transaction handling
8. **Hotel Booking** - Reservations
9. **Movie Ticket Booking** - Seat selection
10. **File System** - Composite pattern

---

## Template for LLD Answer

```markdown
## 1. Requirements & Clarifications
- List assumptions
- Define scope

## 2. Use Cases
- Actor: User
  - Park vehicle
  - Pay for parking
  - Leave

## 3. Class Diagram
[Draw or describe classes and relationships]

## 4. Core Classes
```python
class ParkingLot:
    def park(vehicle): ...
    def leave(ticket): ...

class Vehicle:
    def get_type(): ...
```

## 5. Key Algorithms
- Find available spot
- Calculate fee

## 6. Design Patterns Used
- Factory for vehicles
- Strategy for pricing

## 7. Extensibility
- Adding new vehicle types
- Multiple payment methods
```
