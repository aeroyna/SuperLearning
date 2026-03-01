# Movie Ticket Booking System Design

## Problem Statement

Design an online movie ticket booking system like BookMyShow.

---

## Requirements

### Functional
- Browse movies by city, theater, timing
- View seat availability
- Book tickets with seat selection
- Handle concurrent bookings
- Support multiple payment methods
- Cancel bookings and process refunds

### Non-Functional
- High availability
- Handle concurrent seat selection
- ACID properties for bookings
- Scalable for high traffic

---

## Core Classes

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import List, Dict, Optional, Set
from dataclasses import dataclass, field
from datetime import datetime, timedelta
import threading
import uuid

class SeatType(Enum):
    REGULAR = "regular"
    PREMIUM = "premium"
    VIP = "vip"

class SeatStatus(Enum):
    AVAILABLE = "available"
    SELECTED = "selected"
    BOOKED = "booked"
    BLOCKED = "blocked"

class BookingStatus(Enum):
    PENDING = "pending"
    CONFIRMED = "confirmed"
    CANCELLED = "cancelled"
    EXPIRED = "expired"

class PaymentStatus(Enum):
    PENDING = "pending"
    COMPLETED = "completed"
    FAILED = "failed"
    REFUNDED = "refunded"

@dataclass
class Address:
    street: str
    city: str
    state: str
    zip_code: str
```

---

## Movie and Show Classes

```python
@dataclass
class Movie:
    movie_id: str
    title: str
    description: str
    duration_minutes: int
    language: str
    genre: List[str]
    release_date: datetime
    rating: float = 0.0

@dataclass
class Theater:
    theater_id: str
    name: str
    address: Address
    screens: List['Screen'] = field(default_factory=list)

    def add_screen(self, screen: 'Screen') -> None:
        self.screens.append(screen)

@dataclass
class Seat:
    seat_id: str
    row: str
    number: int
    seat_type: SeatType
    price_multiplier: float = 1.0

    def __hash__(self):
        return hash(self.seat_id)

class Screen:
    def __init__(self, screen_id: str, name: str, total_seats: int):
        self.screen_id = screen_id
        self.name = name
        self.seats: Dict[str, Seat] = {}
        self._setup_seats(total_seats)

    def _setup_seats(self, total_seats: int) -> None:
        rows = "ABCDEFGHIJ"
        seats_per_row = total_seats // len(rows)

        for row_idx, row in enumerate(rows):
            for num in range(1, seats_per_row + 1):
                seat_id = f"{row}{num}"
                seat_type = SeatType.VIP if row in "AB" else (
                    SeatType.PREMIUM if row in "CD" else SeatType.REGULAR
                )
                price_mult = {SeatType.VIP: 1.5, SeatType.PREMIUM: 1.2, SeatType.REGULAR: 1.0}
                self.seats[seat_id] = Seat(
                    seat_id=seat_id,
                    row=row,
                    number=num,
                    seat_type=seat_type,
                    price_multiplier=price_mult[seat_type]
                )

    def get_seat(self, seat_id: str) -> Optional[Seat]:
        return self.seats.get(seat_id)

@dataclass
class Show:
    show_id: str
    movie: Movie
    screen: Screen
    theater: Theater
    start_time: datetime
    end_time: datetime
    base_price: float

    def __post_init__(self):
        self.seat_status: Dict[str, SeatStatus] = {
            seat_id: SeatStatus.AVAILABLE
            for seat_id in self.screen.seats
        }
        self._lock = threading.Lock()
        self._held_seats: Dict[str, datetime] = {}  # seat_id -> hold_expiry

    def get_available_seats(self) -> List[Seat]:
        self._cleanup_expired_holds()
        return [
            self.screen.seats[seat_id]
            for seat_id, status in self.seat_status.items()
            if status == SeatStatus.AVAILABLE
        ]

    def get_seat_price(self, seat: Seat) -> float:
        return self.base_price * seat.price_multiplier

    def hold_seats(self, seat_ids: List[str], duration_seconds: int = 300) -> bool:
        """Temporarily hold seats for booking (with timeout)"""
        with self._lock:
            self._cleanup_expired_holds()

            # Check all seats are available
            for seat_id in seat_ids:
                if self.seat_status.get(seat_id) != SeatStatus.AVAILABLE:
                    return False

            # Hold the seats
            expiry = datetime.now() + timedelta(seconds=duration_seconds)
            for seat_id in seat_ids:
                self.seat_status[seat_id] = SeatStatus.SELECTED
                self._held_seats[seat_id] = expiry

            return True

    def confirm_seats(self, seat_ids: List[str]) -> bool:
        """Confirm held seats as booked"""
        with self._lock:
            for seat_id in seat_ids:
                if seat_id not in self._held_seats:
                    return False
                self.seat_status[seat_id] = SeatStatus.BOOKED
                del self._held_seats[seat_id]
            return True

    def release_seats(self, seat_ids: List[str]) -> None:
        """Release held seats back to available"""
        with self._lock:
            for seat_id in seat_ids:
                if seat_id in self._held_seats:
                    del self._held_seats[seat_id]
                if self.seat_status.get(seat_id) == SeatStatus.SELECTED:
                    self.seat_status[seat_id] = SeatStatus.AVAILABLE

    def _cleanup_expired_holds(self) -> None:
        """Release expired seat holds"""
        now = datetime.now()
        expired = [
            seat_id for seat_id, expiry in self._held_seats.items()
            if expiry < now
        ]
        for seat_id in expired:
            del self._held_seats[seat_id]
            if self.seat_status.get(seat_id) == SeatStatus.SELECTED:
                self.seat_status[seat_id] = SeatStatus.AVAILABLE
```

---

## User and Booking Classes

```python
@dataclass
class User:
    user_id: str
    name: str
    email: str
    phone: str
    bookings: List['Booking'] = field(default_factory=list)

@dataclass
class Booking:
    booking_id: str
    user: User
    show: Show
    seats: List[Seat]
    total_amount: float
    status: BookingStatus = BookingStatus.PENDING
    created_at: datetime = field(default_factory=datetime.now)
    payment: Optional['Payment'] = None

    def calculate_total(self) -> float:
        return sum(self.show.get_seat_price(seat) for seat in self.seats)

    def confirm(self, payment: 'Payment') -> bool:
        if self.status != BookingStatus.PENDING:
            return False
        if payment.status != PaymentStatus.COMPLETED:
            return False

        self.payment = payment
        self.status = BookingStatus.CONFIRMED
        return self.show.confirm_seats([s.seat_id for s in self.seats])

    def cancel(self) -> bool:
        if self.status != BookingStatus.CONFIRMED:
            return False

        self.status = BookingStatus.CANCELLED
        self.show.release_seats([s.seat_id for s in self.seats])
        return True

@dataclass
class Payment:
    payment_id: str
    booking_id: str
    amount: float
    method: str
    status: PaymentStatus = PaymentStatus.PENDING
    transaction_id: Optional[str] = None
    created_at: datetime = field(default_factory=datetime.now)
```

---

## Payment Processing

```python
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float, details: Dict) -> Payment:
        pass

    @abstractmethod
    def refund(self, payment: Payment) -> bool:
        pass

class CreditCardProcessor(PaymentProcessor):
    def process_payment(self, amount: float, details: Dict) -> Payment:
        payment = Payment(
            payment_id=str(uuid.uuid4())[:8],
            booking_id=details.get("booking_id", ""),
            amount=amount,
            method="credit_card"
        )

        # Simulate payment processing
        card_number = details.get("card_number", "")
        if card_number and len(card_number) >= 16:
            payment.status = PaymentStatus.COMPLETED
            payment.transaction_id = f"CC_{uuid.uuid4().hex[:12]}"
        else:
            payment.status = PaymentStatus.FAILED

        return payment

    def refund(self, payment: Payment) -> bool:
        if payment.status == PaymentStatus.COMPLETED:
            payment.status = PaymentStatus.REFUNDED
            return True
        return False

class UPIProcessor(PaymentProcessor):
    def process_payment(self, amount: float, details: Dict) -> Payment:
        payment = Payment(
            payment_id=str(uuid.uuid4())[:8],
            booking_id=details.get("booking_id", ""),
            amount=amount,
            method="upi"
        )

        upi_id = details.get("upi_id", "")
        if upi_id and "@" in upi_id:
            payment.status = PaymentStatus.COMPLETED
            payment.transaction_id = f"UPI_{uuid.uuid4().hex[:12]}"
        else:
            payment.status = PaymentStatus.FAILED

        return payment

    def refund(self, payment: Payment) -> bool:
        if payment.status == PaymentStatus.COMPLETED:
            payment.status = PaymentStatus.REFUNDED
            return True
        return False
```

---

## Booking Service

```python
class BookingService:
    def __init__(self):
        self.movies: Dict[str, Movie] = {}
        self.theaters: Dict[str, Theater] = {}
        self.shows: Dict[str, Show] = {}
        self.bookings: Dict[str, Booking] = {}
        self.users: Dict[str, User] = {}
        self._lock = threading.Lock()

    def add_movie(self, movie: Movie) -> None:
        self.movies[movie.movie_id] = movie

    def add_theater(self, theater: Theater) -> None:
        self.theaters[theater.theater_id] = theater

    def add_show(self, show: Show) -> None:
        self.shows[show.show_id] = show

    def register_user(self, name: str, email: str, phone: str) -> User:
        user_id = f"U{len(self.users) + 1:04d}"
        user = User(user_id, name, email, phone)
        self.users[user_id] = user
        return user

    def search_movies(self, city: str = None, date: datetime = None,
                     genre: str = None) -> List[Movie]:
        """Search movies with filters"""
        results = []
        for show in self.shows.values():
            movie = show.movie
            if city and show.theater.address.city.lower() != city.lower():
                continue
            if date and show.start_time.date() != date.date():
                continue
            if genre and genre.lower() not in [g.lower() for g in movie.genre]:
                continue
            if movie not in results:
                results.append(movie)
        return results

    def get_shows_for_movie(self, movie_id: str, city: str = None,
                           date: datetime = None) -> List[Show]:
        """Get shows for a specific movie"""
        results = []
        for show in self.shows.values():
            if show.movie.movie_id != movie_id:
                continue
            if city and show.theater.address.city.lower() != city.lower():
                continue
            if date and show.start_time.date() != date.date():
                continue
            results.append(show)
        return sorted(results, key=lambda s: s.start_time)

    def start_booking(self, user_id: str, show_id: str,
                     seat_ids: List[str]) -> Optional[Booking]:
        """Start a booking by holding seats"""
        with self._lock:
            user = self.users.get(user_id)
            show = self.shows.get(show_id)

            if not user or not show:
                return None

            # Try to hold seats
            if not show.hold_seats(seat_ids):
                print("Some seats are not available")
                return None

            # Create pending booking
            seats = [show.screen.get_seat(sid) for sid in seat_ids]
            booking = Booking(
                booking_id=f"B{len(self.bookings) + 1:06d}",
                user=user,
                show=show,
                seats=seats,
                total_amount=sum(show.get_seat_price(s) for s in seats)
            )

            self.bookings[booking.booking_id] = booking
            return booking

    def complete_booking(self, booking_id: str,
                        payment_processor: PaymentProcessor,
                        payment_details: Dict) -> bool:
        """Complete a booking with payment"""
        booking = self.bookings.get(booking_id)
        if not booking or booking.status != BookingStatus.PENDING:
            return False

        # Process payment
        payment_details["booking_id"] = booking_id
        payment = payment_processor.process_payment(
            booking.total_amount,
            payment_details
        )

        if payment.status == PaymentStatus.COMPLETED:
            booking.confirm(payment)
            booking.user.bookings.append(booking)
            print(f"Booking confirmed: {booking.booking_id}")
            return True
        else:
            # Release seats if payment failed
            booking.show.release_seats([s.seat_id for s in booking.seats])
            booking.status = BookingStatus.EXPIRED
            print("Payment failed")
            return False

    def cancel_booking(self, booking_id: str) -> float:
        """Cancel a booking and process refund"""
        booking = self.bookings.get(booking_id)
        if not booking or booking.status != BookingStatus.CONFIRMED:
            return 0.0

        # Calculate refund based on time until show
        hours_until_show = (booking.show.start_time - datetime.now()).total_seconds() / 3600
        if hours_until_show < 2:
            refund_percent = 0.0
        elif hours_until_show < 24:
            refund_percent = 0.5
        else:
            refund_percent = 0.9

        refund_amount = booking.total_amount * refund_percent

        # Process refund
        if booking.payment:
            processor = CreditCardProcessor()  # Or get appropriate processor
            processor.refund(booking.payment)

        booking.cancel()
        print(f"Booking cancelled. Refund: ${refund_amount:.2f}")
        return refund_amount

    def get_booking_details(self, booking_id: str) -> Optional[Dict]:
        """Get detailed booking information"""
        booking = self.bookings.get(booking_id)
        if not booking:
            return None

        return {
            "booking_id": booking.booking_id,
            "movie": booking.show.movie.title,
            "theater": booking.show.theater.name,
            "screen": booking.show.screen.name,
            "show_time": booking.show.start_time.strftime("%Y-%m-%d %H:%M"),
            "seats": [f"{s.row}{s.number}" for s in booking.seats],
            "total": f"${booking.total_amount:.2f}",
            "status": booking.status.value
        }
```

---

## Usage Example

```python
def demo_booking_system():
    service = BookingService()

    # Setup theater and screen
    address = Address("123 Main St", "New York", "NY", "10001")
    theater = Theater("T1", "Cineplex Downtown", address)
    screen = Screen("S1", "Screen 1", 100)
    theater.add_screen(screen)
    service.add_theater(theater)

    # Add movie
    movie = Movie(
        movie_id="M1",
        title="The Matrix Resurrections",
        description="Neo returns...",
        duration_minutes=148,
        language="English",
        genre=["Sci-Fi", "Action"],
        release_date=datetime(2021, 12, 22)
    )
    service.add_movie(movie)

    # Add show
    show = Show(
        show_id="SH1",
        movie=movie,
        screen=screen,
        theater=theater,
        start_time=datetime.now() + timedelta(hours=3),
        end_time=datetime.now() + timedelta(hours=5, minutes=28),
        base_price=15.00
    )
    service.add_show(show)

    # Register user
    user = service.register_user("John Doe", "john@email.com", "555-1234")

    # Search movies
    print("=== Available Movies ===")
    movies = service.search_movies(city="New York")
    for m in movies:
        print(f"  {m.title} ({m.language})")

    # Get shows
    print("\n=== Shows for The Matrix ===")
    shows = service.get_shows_for_movie("M1")
    for s in shows:
        print(f"  {s.theater.name} at {s.start_time.strftime('%H:%M')}")

    # View available seats
    print("\n=== Available Seats ===")
    available = show.get_available_seats()
    print(f"  {len(available)} seats available")

    # Start booking
    print("\n=== Booking Flow ===")
    booking = service.start_booking(user.user_id, "SH1", ["A1", "A2"])
    if booking:
        print(f"Booking started: {booking.booking_id}")
        print(f"Total amount: ${booking.total_amount:.2f}")

        # Complete payment
        success = service.complete_booking(
            booking.booking_id,
            CreditCardProcessor(),
            {"card_number": "4111111111111111"}
        )

        if success:
            details = service.get_booking_details(booking.booking_id)
            print(f"Booking Details: {details}")

if __name__ == "__main__":
    demo_booking_system()
```

---

## Design Patterns Used

| Pattern | Usage |
|---------|-------|
| **Strategy** | Payment processors |
| **State** | Booking status management |
| **Factory** | Creating payments |
| **Observer** | Notification on booking |
| **Singleton** | Booking service |

---

## Concurrency Handling

- **Seat holding**: Temporary locks with timeout
- **Optimistic locking**: Check seat availability at confirmation
- **Thread-safe operations**: Using locks for critical sections

---

**Tags**: #lld #case-study #booking #ticket #concurrency
