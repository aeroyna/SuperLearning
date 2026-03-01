# Design Parking Lot

A classic LLD interview problem that tests object-oriented design skills.

---

## 1. Requirements

### Functional
- Multiple levels in the parking lot
- Multiple types of parking spots (compact, large, motorcycle)
- Multiple types of vehicles (car, truck, motorcycle)
- Track available spots
- Issue ticket on entry
- Calculate fee on exit

### Non-Functional
- Handle concurrent entry/exit
- Extensible for new vehicle types
- Different pricing strategies

---

## 2. Use Cases

```
Actor: Driver
1. Enter parking lot
2. Park vehicle in appropriate spot
3. Receive ticket
4. Pay fee
5. Exit parking lot

Actor: Admin
1. View available spots
2. View parking history
3. Configure pricing
```

---

## 3. Class Design

### Core Classes

```python
from abc import ABC, abstractmethod
from enum import Enum
from datetime import datetime
from typing import List, Optional

# Enums
class VehicleSize(Enum):
    MOTORCYCLE = 1
    COMPACT = 2
    LARGE = 3

class SpotStatus(Enum):
    AVAILABLE = "available"
    OCCUPIED = "occupied"
    RESERVED = "reserved"


# Abstract Vehicle
class Vehicle(ABC):
    def __init__(self, license_plate: str):
        self.license_plate = license_plate

    @abstractmethod
    def get_size(self) -> VehicleSize:
        pass


class Motorcycle(Vehicle):
    def get_size(self) -> VehicleSize:
        return VehicleSize.MOTORCYCLE


class Car(Vehicle):
    def get_size(self) -> VehicleSize:
        return VehicleSize.COMPACT


class Truck(Vehicle):
    def get_size(self) -> VehicleSize:
        return VehicleSize.LARGE


# Parking Spot
class ParkingSpot:
    def __init__(self, spot_id: str, size: VehicleSize, level: int):
        self.spot_id = spot_id
        self.size = size
        self.level = level
        self.status = SpotStatus.AVAILABLE
        self.vehicle: Optional[Vehicle] = None

    def can_fit(self, vehicle: Vehicle) -> bool:
        return (
            self.status == SpotStatus.AVAILABLE and
            vehicle.get_size().value <= self.size.value
        )

    def assign_vehicle(self, vehicle: Vehicle):
        self.vehicle = vehicle
        self.status = SpotStatus.OCCUPIED

    def remove_vehicle(self) -> Vehicle:
        vehicle = self.vehicle
        self.vehicle = None
        self.status = SpotStatus.AVAILABLE
        return vehicle


# Level
class Level:
    def __init__(self, floor: int, spots: List[ParkingSpot]):
        self.floor = floor
        self.spots = spots

    def find_available_spot(self, vehicle: Vehicle) -> Optional[ParkingSpot]:
        for spot in self.spots:
            if spot.can_fit(vehicle):
                return spot
        return None

    def get_available_count(self, size: VehicleSize = None) -> int:
        return sum(
            1 for spot in self.spots
            if spot.status == SpotStatus.AVAILABLE and
               (size is None or spot.size == size)
        )
```

### Ticket and Payment

```python
class Ticket:
    def __init__(self, vehicle: Vehicle, spot: ParkingSpot):
        self.ticket_id = self._generate_id()
        self.vehicle = vehicle
        self.spot = spot
        self.entry_time = datetime.now()
        self.exit_time: Optional[datetime] = None
        self.is_paid = False

    def _generate_id(self) -> str:
        import uuid
        return str(uuid.uuid4())[:8]


# Strategy Pattern for Pricing
class PricingStrategy(ABC):
    @abstractmethod
    def calculate_fee(self, ticket: Ticket) -> float:
        pass


class HourlyPricing(PricingStrategy):
    def __init__(self, rate_per_hour: float):
        self.rate = rate_per_hour

    def calculate_fee(self, ticket: Ticket) -> float:
        duration = (ticket.exit_time - ticket.entry_time).total_seconds() / 3600
        hours = max(1, int(duration) + (1 if duration % 1 > 0 else 0))
        return hours * self.rate


class FlatRatePricing(PricingStrategy):
    def __init__(self, flat_rate: float):
        self.rate = flat_rate

    def calculate_fee(self, ticket: Ticket) -> float:
        return self.rate


class Payment:
    def __init__(self, ticket: Ticket, amount: float):
        self.ticket = ticket
        self.amount = amount
        self.payment_time = datetime.now()
        self.is_successful = False

    def process(self) -> bool:
        # Payment processing logic
        self.is_successful = True
        self.ticket.is_paid = True
        return True
```

### Parking Lot (Main Class)

```python
import threading

class ParkingLot:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self, name: str, levels: List[Level], pricing: PricingStrategy):
        self.name = name
        self.levels = levels
        self.pricing = pricing
        self.active_tickets: dict[str, Ticket] = {}
        self._spot_lock = threading.Lock()

    def park_vehicle(self, vehicle: Vehicle) -> Optional[Ticket]:
        with self._spot_lock:
            spot = self._find_available_spot(vehicle)
            if spot is None:
                return None

            spot.assign_vehicle(vehicle)
            ticket = Ticket(vehicle, spot)
            self.active_tickets[ticket.ticket_id] = ticket
            return ticket

    def _find_available_spot(self, vehicle: Vehicle) -> Optional[ParkingSpot]:
        for level in self.levels:
            spot = level.find_available_spot(vehicle)
            if spot:
                return spot
        return None

    def exit_vehicle(self, ticket_id: str) -> Optional[Payment]:
        ticket = self.active_tickets.get(ticket_id)
        if ticket is None:
            return None

        ticket.exit_time = datetime.now()
        fee = self.pricing.calculate_fee(ticket)

        payment = Payment(ticket, fee)
        if payment.process():
            ticket.spot.remove_vehicle()
            del self.active_tickets[ticket_id]
            return payment

        return None

    def get_available_spots(self, size: VehicleSize = None) -> int:
        return sum(level.get_available_count(size) for level in self.levels)
```

---

## 4. Entry/Exit Panels

```python
class EntryPanel:
    def __init__(self, panel_id: str, parking_lot: ParkingLot):
        self.panel_id = panel_id
        self.parking_lot = parking_lot

    def get_ticket(self, vehicle: Vehicle) -> Optional[Ticket]:
        ticket = self.parking_lot.park_vehicle(vehicle)
        if ticket:
            self._print_ticket(ticket)
        return ticket

    def _print_ticket(self, ticket: Ticket):
        print(f"Ticket: {ticket.ticket_id}")
        print(f"Spot: {ticket.spot.spot_id}")
        print(f"Entry: {ticket.entry_time}")


class ExitPanel:
    def __init__(self, panel_id: str, parking_lot: ParkingLot):
        self.panel_id = panel_id
        self.parking_lot = parking_lot

    def process_exit(self, ticket_id: str) -> bool:
        payment = self.parking_lot.exit_vehicle(ticket_id)
        if payment:
            print(f"Amount: ${payment.amount:.2f}")
            return True
        return False
```

---

## 5. Usage Example

```python
# Create parking lot
spots_level1 = [
    ParkingSpot(f"1-{i}", VehicleSize.COMPACT, 1) for i in range(10)
] + [
    ParkingSpot(f"1-M{i}", VehicleSize.MOTORCYCLE, 1) for i in range(5)
]

spots_level2 = [
    ParkingSpot(f"2-{i}", VehicleSize.LARGE, 2) for i in range(5)
]

levels = [
    Level(1, spots_level1),
    Level(2, spots_level2)
]

parking_lot = ParkingLot("Downtown Parking", levels, HourlyPricing(5.0))

# Driver flow
entry = EntryPanel("E1", parking_lot)
exit_panel = ExitPanel("X1", parking_lot)

car = Car("ABC-123")
ticket = entry.get_ticket(car)

print(f"Available spots: {parking_lot.get_available_spots()}")

# Later...
exit_panel.process_exit(ticket.ticket_id)
```

---

## 6. Design Patterns Used

| Pattern | Application |
|---------|-------------|
| Singleton | ParkingLot - one instance |
| Factory | VehicleFactory for creating vehicles |
| Strategy | PricingStrategy for different pricing models |
| Observer | Notify when spots become available |

---

## 7. Extensibility

### Add Electric Vehicle
```python
class ElectricCar(Car):
    def __init__(self, license_plate: str, battery_level: int):
        super().__init__(license_plate)
        self.battery_level = battery_level

class ChargingSpot(ParkingSpot):
    def __init__(self, spot_id: str, level: int):
        super().__init__(spot_id, VehicleSize.COMPACT, level)
        self.is_charging = False
```

### Add Reservation
```python
class Reservation:
    def __init__(self, spot: ParkingSpot, start: datetime, end: datetime):
        self.spot = spot
        self.start = start
        self.end = end
        spot.status = SpotStatus.RESERVED
```

---

## 8. Thread Safety

```python
class ParkingLot:
    def park_vehicle(self, vehicle: Vehicle) -> Optional[Ticket]:
        with self._spot_lock:  # Thread-safe
            spot = self._find_available_spot(vehicle)
            if spot is None:
                return None
            spot.assign_vehicle(vehicle)
            # ...
```
