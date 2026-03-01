# Library Management System Design

## Problem Statement

Design a library management system that handles book inventory, member management, and lending operations.

---

## Requirements

### Functional
- Add, update, remove books
- Register and manage members
- Issue and return books
- Search books by title, author, ISBN
- Track overdue books and calculate fines
- Reserve books that are currently borrowed

### Non-Functional
- Support concurrent access
- Maintain data integrity
- Generate reports
- Handle multiple copies of same book

---

## Core Classes

```python
from abc import ABC, abstractmethod
from enum import Enum
from typing import List, Dict, Optional
from dataclasses import dataclass, field
from datetime import datetime, timedelta
import uuid

class BookStatus(Enum):
    AVAILABLE = "available"
    BORROWED = "borrowed"
    RESERVED = "reserved"
    LOST = "lost"

class MemberStatus(Enum):
    ACTIVE = "active"
    SUSPENDED = "suspended"
    EXPIRED = "expired"

class AccountType(Enum):
    STANDARD = "standard"
    PREMIUM = "premium"
    STUDENT = "student"

@dataclass
class Address:
    street: str
    city: str
    state: str
    zip_code: str
```

---

## Book Classes

```python
@dataclass
class Author:
    name: str
    biography: str = ""

@dataclass
class Book:
    """Represents a book title (not a physical copy)"""
    isbn: str
    title: str
    authors: List[Author]
    publisher: str
    publication_year: int
    subject: str
    pages: int = 0

    def __hash__(self):
        return hash(self.isbn)

@dataclass
class BookItem:
    """Represents a physical copy of a book"""
    barcode: str
    book: Book
    status: BookStatus = BookStatus.AVAILABLE
    date_purchased: datetime = field(default_factory=datetime.now)
    rack_location: str = ""
    price: float = 0.0

    borrowed_date: Optional[datetime] = None
    due_date: Optional[datetime] = None
    borrowed_by: Optional[str] = None  # Member ID

    def is_available(self) -> bool:
        return self.status == BookStatus.AVAILABLE

    def checkout(self, member_id: str, days: int = 14) -> None:
        self.status = BookStatus.BORROWED
        self.borrowed_date = datetime.now()
        self.due_date = datetime.now() + timedelta(days=days)
        self.borrowed_by = member_id

    def return_book(self) -> None:
        self.status = BookStatus.AVAILABLE
        self.borrowed_date = None
        self.due_date = None
        self.borrowed_by = None

    def is_overdue(self) -> bool:
        if self.due_date:
            return datetime.now() > self.due_date
        return False

    def get_overdue_days(self) -> int:
        if self.is_overdue():
            return (datetime.now() - self.due_date).days
        return 0
```

---

## Member Classes

```python
@dataclass
class Member:
    member_id: str
    name: str
    email: str
    phone: str
    address: Address
    account_type: AccountType = AccountType.STANDARD
    status: MemberStatus = MemberStatus.ACTIVE
    date_registered: datetime = field(default_factory=datetime.now)

    borrowed_books: List[str] = field(default_factory=list)  # Barcodes
    reserved_books: List[str] = field(default_factory=list)  # ISBNs
    fine_amount: float = 0.0

    @property
    def max_books(self) -> int:
        limits = {
            AccountType.STUDENT: 3,
            AccountType.STANDARD: 5,
            AccountType.PREMIUM: 10
        }
        return limits[self.account_type]

    @property
    def loan_period(self) -> int:
        periods = {
            AccountType.STUDENT: 7,
            AccountType.STANDARD: 14,
            AccountType.PREMIUM: 21
        }
        return periods[self.account_type]

    def can_borrow(self) -> bool:
        return (self.status == MemberStatus.ACTIVE and
                len(self.borrowed_books) < self.max_books and
                self.fine_amount < 10.0)

    def add_borrowed_book(self, barcode: str) -> None:
        self.borrowed_books.append(barcode)

    def remove_borrowed_book(self, barcode: str) -> None:
        if barcode in self.borrowed_books:
            self.borrowed_books.remove(barcode)

    def add_fine(self, amount: float) -> None:
        self.fine_amount += amount

    def pay_fine(self, amount: float) -> float:
        payment = min(amount, self.fine_amount)
        self.fine_amount -= payment
        return payment
```

---

## Reservation System

```python
@dataclass
class Reservation:
    reservation_id: str
    isbn: str
    member_id: str
    reservation_date: datetime = field(default_factory=datetime.now)
    notification_sent: bool = False
    expiry_date: Optional[datetime] = None

    def __post_init__(self):
        if self.expiry_date is None:
            self.expiry_date = self.reservation_date + timedelta(days=3)

    def is_expired(self) -> bool:
        return datetime.now() > self.expiry_date

class ReservationManager:
    def __init__(self):
        self.reservations: Dict[str, List[Reservation]] = {}  # ISBN -> reservations

    def reserve(self, isbn: str, member_id: str) -> Reservation:
        reservation = Reservation(
            reservation_id=str(uuid.uuid4())[:8],
            isbn=isbn,
            member_id=member_id
        )

        if isbn not in self.reservations:
            self.reservations[isbn] = []
        self.reservations[isbn].append(reservation)

        return reservation

    def get_next_reservation(self, isbn: str) -> Optional[Reservation]:
        if isbn in self.reservations:
            # Remove expired reservations
            self.reservations[isbn] = [
                r for r in self.reservations[isbn] if not r.is_expired()
            ]
            if self.reservations[isbn]:
                return self.reservations[isbn][0]
        return None

    def fulfill_reservation(self, isbn: str, member_id: str) -> bool:
        if isbn in self.reservations:
            for i, r in enumerate(self.reservations[isbn]):
                if r.member_id == member_id:
                    self.reservations[isbn].pop(i)
                    return True
        return False

    def cancel_reservation(self, reservation_id: str) -> bool:
        for isbn, reservations in self.reservations.items():
            for i, r in enumerate(reservations):
                if r.reservation_id == reservation_id:
                    reservations.pop(i)
                    return True
        return False
```

---

## Library Catalog

```python
class Catalog:
    """Manages book inventory and search"""

    def __init__(self):
        self.books: Dict[str, Book] = {}  # ISBN -> Book
        self.book_items: Dict[str, BookItem] = {}  # Barcode -> BookItem
        self.isbn_to_barcodes: Dict[str, List[str]] = {}  # ISBN -> Barcodes

    def add_book(self, book: Book) -> None:
        self.books[book.isbn] = book
        if book.isbn not in self.isbn_to_barcodes:
            self.isbn_to_barcodes[book.isbn] = []

    def add_book_item(self, book_item: BookItem) -> None:
        self.book_items[book_item.barcode] = book_item
        isbn = book_item.book.isbn
        if isbn not in self.isbn_to_barcodes:
            self.isbn_to_barcodes[isbn] = []
        self.isbn_to_barcodes[isbn].append(book_item.barcode)

    def remove_book_item(self, barcode: str) -> bool:
        if barcode in self.book_items:
            book_item = self.book_items[barcode]
            isbn = book_item.book.isbn
            self.isbn_to_barcodes[isbn].remove(barcode)
            del self.book_items[barcode]
            return True
        return False

    def get_available_copies(self, isbn: str) -> List[BookItem]:
        barcodes = self.isbn_to_barcodes.get(isbn, [])
        return [
            self.book_items[bc] for bc in barcodes
            if self.book_items[bc].is_available()
        ]

    def search_by_title(self, title: str) -> List[Book]:
        title_lower = title.lower()
        return [
            book for book in self.books.values()
            if title_lower in book.title.lower()
        ]

    def search_by_author(self, author_name: str) -> List[Book]:
        author_lower = author_name.lower()
        results = []
        for book in self.books.values():
            for author in book.authors:
                if author_lower in author.name.lower():
                    results.append(book)
                    break
        return results

    def search_by_isbn(self, isbn: str) -> Optional[Book]:
        return self.books.get(isbn)

    def search_by_subject(self, subject: str) -> List[Book]:
        subject_lower = subject.lower()
        return [
            book for book in self.books.values()
            if subject_lower in book.subject.lower()
        ]

    def get_total_copies(self, isbn: str) -> int:
        return len(self.isbn_to_barcodes.get(isbn, []))

    def get_available_count(self, isbn: str) -> int:
        return len(self.get_available_copies(isbn))
```

---

## Library System

```python
class LibrarySystem:
    """Main system coordinating all library operations"""

    FINE_PER_DAY = 0.50

    def __init__(self):
        self.catalog = Catalog()
        self.members: Dict[str, Member] = {}
        self.reservation_manager = ReservationManager()
        self.transaction_log: List[Dict] = []

    def register_member(self, name: str, email: str, phone: str,
                       address: Address,
                       account_type: AccountType = AccountType.STANDARD) -> Member:
        member_id = f"M{len(self.members) + 1:04d}"
        member = Member(
            member_id=member_id,
            name=name,
            email=email,
            phone=phone,
            address=address,
            account_type=account_type
        )
        self.members[member_id] = member
        self._log_transaction("REGISTER", member_id=member_id)
        return member

    def add_book(self, book: Book, num_copies: int = 1) -> List[BookItem]:
        self.catalog.add_book(book)
        items = []
        for i in range(num_copies):
            barcode = f"{book.isbn}-{i+1:03d}"
            item = BookItem(barcode=barcode, book=book)
            self.catalog.add_book_item(item)
            items.append(item)
        return items

    def checkout_book(self, member_id: str, barcode: str) -> bool:
        """Issue a book to a member"""
        if member_id not in self.members:
            print(f"Member {member_id} not found")
            return False

        member = self.members[member_id]
        if not member.can_borrow():
            print(f"Member cannot borrow (status: {member.status}, "
                  f"books: {len(member.borrowed_books)}, fines: ${member.fine_amount})")
            return False

        if barcode not in self.catalog.book_items:
            print(f"Book {barcode} not found")
            return False

        book_item = self.catalog.book_items[barcode]
        if not book_item.is_available():
            print(f"Book {barcode} not available")
            return False

        # Check if there's a reservation for someone else
        isbn = book_item.book.isbn
        next_reservation = self.reservation_manager.get_next_reservation(isbn)
        if next_reservation and next_reservation.member_id != member_id:
            print(f"Book is reserved for another member")
            return False

        # Checkout
        book_item.checkout(member_id, member.loan_period)
        member.add_borrowed_book(barcode)

        # Fulfill reservation if any
        if next_reservation:
            self.reservation_manager.fulfill_reservation(isbn, member_id)

        self._log_transaction("CHECKOUT", member_id=member_id, barcode=barcode)
        print(f"Book checked out. Due date: {book_item.due_date.date()}")
        return True

    def return_book(self, barcode: str) -> float:
        """Return a book and calculate any fines"""
        if barcode not in self.catalog.book_items:
            print(f"Book {barcode} not found")
            return 0.0

        book_item = self.catalog.book_items[barcode]
        if book_item.status != BookStatus.BORROWED:
            print(f"Book {barcode} was not borrowed")
            return 0.0

        member_id = book_item.borrowed_by
        member = self.members.get(member_id)

        # Calculate fine
        fine = 0.0
        if book_item.is_overdue():
            overdue_days = book_item.get_overdue_days()
            fine = overdue_days * self.FINE_PER_DAY
            if member:
                member.add_fine(fine)
            print(f"Book was {overdue_days} days overdue. Fine: ${fine:.2f}")

        # Return book
        book_item.return_book()
        if member:
            member.remove_borrowed_book(barcode)

        # Notify next reservation
        isbn = book_item.book.isbn
        next_reservation = self.reservation_manager.get_next_reservation(isbn)
        if next_reservation:
            self._notify_reservation(next_reservation)

        self._log_transaction("RETURN", member_id=member_id, barcode=barcode, fine=fine)
        return fine

    def reserve_book(self, member_id: str, isbn: str) -> Optional[Reservation]:
        """Reserve a book that's currently unavailable"""
        if member_id not in self.members:
            return None

        if isbn not in self.catalog.books:
            return None

        # Check if book is already available
        available = self.catalog.get_available_copies(isbn)
        if available:
            print("Book is currently available, no need to reserve")
            return None

        reservation = self.reservation_manager.reserve(isbn, member_id)
        self.members[member_id].reserved_books.append(isbn)
        self._log_transaction("RESERVE", member_id=member_id, isbn=isbn)
        print(f"Book reserved. Position in queue: "
              f"{len(self.reservation_manager.reservations.get(isbn, []))}")
        return reservation

    def renew_book(self, barcode: str) -> bool:
        """Renew a borrowed book"""
        if barcode not in self.catalog.book_items:
            return False

        book_item = self.catalog.book_items[barcode]
        if book_item.status != BookStatus.BORROWED:
            return False

        # Check if book has reservation
        isbn = book_item.book.isbn
        if self.reservation_manager.get_next_reservation(isbn):
            print("Cannot renew - book has reservations")
            return False

        # Cannot renew if overdue
        if book_item.is_overdue():
            print("Cannot renew - book is overdue")
            return False

        member = self.members.get(book_item.borrowed_by)
        if member:
            book_item.due_date = datetime.now() + timedelta(days=member.loan_period)
            print(f"Book renewed. New due date: {book_item.due_date.date()}")
            return True
        return False

    def pay_fine(self, member_id: str, amount: float) -> float:
        """Pay member's fines"""
        if member_id not in self.members:
            return 0.0

        member = self.members[member_id]
        paid = member.pay_fine(amount)
        self._log_transaction("PAYMENT", member_id=member_id, amount=paid)
        return paid

    def get_overdue_books(self) -> List[BookItem]:
        """Get all overdue books"""
        return [
            item for item in self.catalog.book_items.values()
            if item.is_overdue()
        ]

    def get_member_books(self, member_id: str) -> List[BookItem]:
        """Get all books borrowed by a member"""
        if member_id not in self.members:
            return []

        member = self.members[member_id]
        return [
            self.catalog.book_items[bc]
            for bc in member.borrowed_books
            if bc in self.catalog.book_items
        ]

    def _notify_reservation(self, reservation: Reservation) -> None:
        """Notify member that reserved book is available"""
        print(f"Notifying {reservation.member_id}: Reserved book is available")
        reservation.notification_sent = True

    def _log_transaction(self, action: str, **kwargs) -> None:
        """Log a transaction"""
        self.transaction_log.append({
            "timestamp": datetime.now(),
            "action": action,
            **kwargs
        })
```

---

## Usage Example

```python
def demo_library_system():
    library = LibrarySystem()

    # Add books
    book1 = Book(
        isbn="978-0-13-468599-1",
        title="Clean Code",
        authors=[Author("Robert C. Martin")],
        publisher="Prentice Hall",
        publication_year=2008,
        subject="Programming"
    )

    book2 = Book(
        isbn="978-0-201-63361-0",
        title="Design Patterns",
        authors=[
            Author("Erich Gamma"),
            Author("Richard Helm"),
            Author("Ralph Johnson"),
            Author("John Vlissides")
        ],
        publisher="Addison-Wesley",
        publication_year=1994,
        subject="Software Design"
    )

    library.add_book(book1, num_copies=3)
    library.add_book(book2, num_copies=2)

    # Register members
    address = Address("123 Main St", "Springfield", "IL", "62701")
    member1 = library.register_member("John Doe", "john@email.com",
                                       "555-1234", address)
    member2 = library.register_member("Jane Smith", "jane@email.com",
                                       "555-5678", address, AccountType.PREMIUM)

    # Search books
    print("\n=== Search Results ===")
    results = library.catalog.search_by_title("Clean")
    for book in results:
        print(f"Found: {book.title} by {book.authors[0].name}")

    # Checkout books
    print("\n=== Checkout ===")
    library.checkout_book(member1.member_id, "978-0-13-468599-1-001")
    library.checkout_book(member1.member_id, "978-0-201-63361-0-001")

    # Try to reserve an unavailable book
    print("\n=== Reserve ===")
    library.checkout_book(member2.member_id, "978-0-13-468599-1-002")
    library.checkout_book(member2.member_id, "978-0-13-468599-1-003")
    # All copies borrowed, now reserve
    library.reserve_book(member2.member_id, "978-0-13-468599-1")

    # Return a book
    print("\n=== Return ===")
    library.return_book("978-0-13-468599-1-001")

    # Renew a book
    print("\n=== Renew ===")
    library.renew_book("978-0-201-63361-0-001")

    # Check member's borrowed books
    print("\n=== Member Books ===")
    for book_item in library.get_member_books(member1.member_id):
        print(f"{book_item.book.title} - Due: {book_item.due_date.date()}")

if __name__ == "__main__":
    demo_library_system()
```

---

## Design Patterns Used

| Pattern | Usage |
|---------|-------|
| **Strategy** | Different search algorithms |
| **Observer** | Notifications for reservations |
| **Factory** | Creating members and book items |
| **Repository** | Catalog for data access |

---

**Tags**: #lld #case-study #library #catalog #reservation
