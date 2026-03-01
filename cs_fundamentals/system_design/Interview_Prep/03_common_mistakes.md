# Common Mistakes in System Design Interviews

Avoid these pitfalls to maximize your interview performance.

---

## HLD Mistakes

### 1. Jumping Straight to Design

```
❌ WRONG
"Let me start drawing the architecture..."

✓ CORRECT
"Before I design, I'd like to clarify a few things:
 - What's our expected scale?
 - Which features should I prioritize?
 - Are there any specific constraints?"
```

**Why it matters:** Without requirements, you might design for the wrong scale or miss critical features. Interviewers want to see how you gather information.

---

### 2. Ignoring Scale Estimation

```
❌ WRONG
"We'll use a database to store the data."

✓ CORRECT
"With 100M DAU making 10 requests/day, we have:
 - 1B requests/day → ~12K QPS average, 36K peak
 - If each record is 1KB, that's 1TB/day
 - Over 3 years: 1 PB storage

 At this scale, we need sharding and caching."
```

**Why it matters:** Numbers drive architectural decisions. A system for 1K users differs vastly from one for 1B users.

---

### 3. Single Point of Failure

```
❌ WRONG
┌────────┐     ┌──────────┐     ┌──────────┐
│ Client │────▶│  Server  │────▶│ Database │
└────────┘     └──────────┘     └──────────┘
                    ↑
               Single server = SPOF

✓ CORRECT
┌────────┐     ┌──────────────┐     ┌─────────────────┐
│ Client │────▶│ Load Balancer│────▶│  Server Pool    │
└────────┘     │   (Active/   │     │  (3+ servers)   │
               │   Standby)   │     └────────┬────────┘
               └──────────────┘              │
                                    ┌────────▼────────┐
                                    │ Database Cluster│
                                    │  (Primary +     │
                                    │   Replicas)     │
                                    └─────────────────┘
```

**Why it matters:** Production systems need redundancy. Always show multiple instances.

---

### 4. Forgetting About Data Consistency

```
❌ WRONG
"We'll replicate data across regions for availability."
(No mention of consistency implications)

✓ CORRECT
"With multi-region replication, we need to choose:
 - Strong consistency: Higher latency, use for payments
 - Eventual consistency: Lower latency, use for feeds

 For this use case, eventual consistency is acceptable
 because users can tolerate seeing slightly stale data."
```

**Why it matters:** CAP theorem trade-offs are fundamental. Show you understand the implications.

---

### 5. Over-Engineering

```
❌ WRONG (for a URL shortener)
"We'll use Kafka for event streaming, Elasticsearch
for analytics, Redis cluster for caching, multiple
microservices, Kubernetes for orchestration..."

✓ CORRECT
"Let's start simple:
 - Monolith with URL service
 - Single PostgreSQL with sharding ready
 - Redis cache for hot URLs

 We can add complexity as needed. For 100K QPS,
 this architecture handles it with room to grow."
```

**Why it matters:** Start simple, add complexity only when justified by requirements.

---

### 6. Not Discussing Trade-offs

```
❌ WRONG
"We'll use NoSQL."

✓ CORRECT
"For database choice, let's consider:

SQL (PostgreSQL):
+ ACID transactions for booking integrity
+ Complex queries for analytics
- Harder to scale horizontally

NoSQL (Cassandra):
+ Excellent write throughput
+ Easy horizontal scaling
- No transactions, eventual consistency

For a booking system, I recommend SQL because
transaction integrity is critical. We can add
read replicas and caching for scale."
```

**Why it matters:** There's no "right" answer—interviewers want to see your reasoning.

---

### 7. Ignoring the Read/Write Ratio

```
❌ WRONG
"Users will read and write data."

✓ CORRECT
"This is a read-heavy system:
 - Reads: 100K QPS (users viewing content)
 - Writes: 100 QPS (users creating content)
 - Ratio: 1000:1

Strategy:
 - Aggressive caching for reads (Redis, CDN)
 - Write to primary, read from replicas
 - Eventually consistent is acceptable"
```

**Why it matters:** Read/write ratio determines your caching and replication strategy.

---

### 8. Missing Monitoring & Alerting

```
❌ WRONG
(No mention of observability)

✓ CORRECT
"For operations, we need:
 - Metrics: QPS, latency p50/p99, error rates
 - Logging: Structured logs with trace IDs
 - Alerting: PagerDuty for critical issues
 - Dashboards: Grafana for visualization

 Key SLOs:
 - Availability: 99.9% (8.7 hours downtime/year)
 - Latency: p99 < 500ms"
```

**Why it matters:** Production systems need observability. Mentioning this shows operational maturity.

---

## LLD Mistakes

### 1. Starting with Code Instead of Design

```
❌ WRONG
"Let me start coding the ParkingLot class..."

✓ CORRECT
"Before coding, let me identify the key objects:
 - ParkingLot: Manages spots and entries
 - ParkingSpot: Represents a single spot
 - Vehicle: Abstract class with Car, Motorcycle
 - Ticket: Tracks parking session
 - PaymentProcessor: Handles payment

Let me sketch the class relationships first."
```

**Why it matters:** Design first, code second. Show your thought process.

---

### 2. Violating Single Responsibility

```python
# ❌ WRONG: God class doing everything
class ParkingLot:
    def park_vehicle(self, vehicle): ...
    def calculate_fee(self, ticket): ...
    def process_payment(self, amount): ...
    def send_receipt_email(self, email): ...
    def update_display_board(self): ...
    def generate_report(self): ...

# ✓ CORRECT: Separated responsibilities
class ParkingLot:
    def park_vehicle(self, vehicle): ...
    def exit_vehicle(self, ticket): ...

class FeeCalculator:
    def calculate(self, ticket) -> Decimal: ...

class PaymentProcessor:
    def process(self, amount) -> Receipt: ...

class NotificationService:
    def send_receipt(self, email, receipt): ...

class DisplayBoard:
    def update(self, available_spots): ...
```

**Why it matters:** SRP is the most important SOLID principle. Each class should have one reason to change.

---

### 3. Using Inheritance When Composition is Better

```python
# ❌ WRONG: Inheritance for code reuse
class FlyingCar(Car, Airplane):  # Diamond problem!
    pass

# ✓ CORRECT: Composition
class Vehicle:
    def __init__(
        self,
        engine: IEngine,
        movement: IMovementStrategy
    ):
        self._engine = engine
        self._movement = movement

    def move(self):
        self._movement.move(self)

# Now we can combine any engine with any movement
flying_car = Vehicle(
    engine=JetEngine(),
    movement=FlyingMovement()
)
```

**Why it matters:** "Favor composition over inheritance" allows flexible combinations without hierarchy issues.

---

### 4. Not Using Interfaces/Abstract Classes

```python
# ❌ WRONG: Direct dependency on concrete class
class PaymentService:
    def __init__(self):
        self._processor = StripeProcessor()  # Tightly coupled!

    def pay(self, amount):
        self._processor.charge(amount)

# ✓ CORRECT: Depend on abstraction
class IPaymentProcessor(ABC):
    @abstractmethod
    def charge(self, amount: Decimal) -> bool: ...

class PaymentService:
    def __init__(self, processor: IPaymentProcessor):
        self._processor = processor  # Injected!

    def pay(self, amount):
        self._processor.charge(amount)

# Now easy to swap: StripeProcessor, PayPalProcessor, MockProcessor
```

**Why it matters:** Dependency inversion enables testing and flexibility.

---

### 5. Ignoring Edge Cases

```python
# ❌ WRONG: No validation
class Elevator:
    def go_to_floor(self, floor: int):
        self._current_floor = floor

# ✓ CORRECT: Handle edge cases
class Elevator:
    def __init__(self, min_floor: int, max_floor: int):
        self._min_floor = min_floor
        self._max_floor = max_floor
        self._current_floor = min_floor

    def go_to_floor(self, floor: int) -> bool:
        if not self._min_floor <= floor <= self._max_floor:
            raise InvalidFloorError(floor, self._min_floor, self._max_floor)

        if floor == self._current_floor:
            return True  # Already there

        if self._is_in_maintenance:
            raise ElevatorUnavailableError("Elevator in maintenance")

        self._move_to(floor)
        return True
```

**Why it matters:** Real systems handle errors gracefully. Show defensive programming.

---

### 6. Hardcoding Instead of Configuration

```python
# ❌ WRONG: Magic numbers everywhere
class ParkingLot:
    def calculate_fee(self, hours):
        return hours * 5  # What is 5?

# ✓ CORRECT: Configurable
@dataclass
class ParkingConfig:
    hourly_rate: Decimal = Decimal("5.00")
    max_hours: int = 24
    free_minutes: int = 15

class ParkingLot:
    def __init__(self, config: ParkingConfig):
        self._config = config

    def calculate_fee(self, duration: timedelta) -> Decimal:
        if duration.total_seconds() < self._config.free_minutes * 60:
            return Decimal("0")
        hours = ceil(duration.total_seconds() / 3600)
        return min(hours, self._config.max_hours) * self._config.hourly_rate
```

**Why it matters:** Configuration allows changing behavior without code changes.

---

### 7. Not Considering Thread Safety

```python
# ❌ WRONG: Race condition in singleton
class ConnectionPool:
    _instance = None

    @classmethod
    def get_instance(cls):
        if cls._instance is None:  # Two threads can both enter here!
            cls._instance = cls()
        return cls._instance

# ✓ CORRECT: Thread-safe singleton
class ConnectionPool:
    _instance = None
    _lock = threading.Lock()

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:  # Double-check
                    cls._instance = cls()
        return cls._instance
```

**Why it matters:** Multi-threaded issues are subtle. Mention when thread safety is needed.

---

### 8. Poor Naming

```python
# ❌ WRONG: Unclear names
class M:
    def p(self, d):
        self.x = d.a * d.b
        return self.x

# ✓ CORRECT: Self-documenting
class PriceCalculator:
    def calculate_total(self, order: Order) -> Decimal:
        self._total = order.quantity * order.unit_price
        return self._total
```

**Why it matters:** Code should read like documentation. Clear names reduce confusion.

---

## General Interview Mistakes

### 1. Staying Silent

```
❌ WRONG
*Thinking quietly for 2 minutes*

✓ CORRECT
"I'm considering two approaches here:
 Option A: Use a message queue for async processing
 Option B: Synchronous API calls with retries

 Let me think about the trade-offs...

 For our use case with high throughput requirements,
 I'll go with Option A because..."
```

**Why it matters:** Interviewers can't evaluate what they can't hear. Think out loud.

---

### 2. Not Managing Time

```
❌ WRONG
Spending 20 minutes on requirements,
leaving no time for deep dive

✓ CORRECT
"We have 45 minutes. I'll spend:
 - 5 min: Requirements and estimation
 - 5 min: API design
 - 15 min: High-level architecture
 - 15 min: Deep dive on key component
 - 5 min: Wrap up and questions"
```

**Why it matters:** Cover all phases. It's better to have a complete design than a perfect partial one.

---

### 3. Arguing with Feedback

```
❌ WRONG
Interviewer: "Have you considered caching?"
"No, caching isn't needed here because..."

✓ CORRECT
Interviewer: "Have you considered caching?"
"That's a good point. Let me add a cache layer:
 - Cache: Redis for session data
 - TTL: 1 hour for user profiles
 - Invalidation: Write-through for consistency

 This reduces database load significantly."
```

**Why it matters:** Hints are gifts. Use them to improve your design.

---

### 4. Not Asking Clarifying Questions

```
❌ WRONG
"Design Twitter."
*Immediately starts designing*

✓ CORRECT
"Design Twitter."
"Before I start, I have a few questions:
 1. Should I focus on the feed, posting, or both?
 2. What's our target scale? 100M or 1B users?
 3. Do we need real-time updates or is polling okay?
 4. Should I consider the recommendation system?"
```

**Why it matters:** Clarifying questions show you understand that requirements drive design.

---

## Quick Reference: Red Flags

| Red Flag | Better Approach |
|----------|-----------------|
| "Let me start coding" | "Let me first understand the requirements" |
| "We'll just add more servers" | "Here's our specific scaling strategy..." |
| "This is the best approach" | "This approach has trade-offs: ..." |
| "I don't know" (and stops) | "I'm not sure, but I'd approach it by..." |
| *Long silence* | "I'm thinking through the options..." |
| "We'll use microservices" | "Given our scale, microservices make sense because..." |
| Ignoring interviewer hints | "That's a good point, let me incorporate that" |
| Over-complicating simple problems | "Let's start simple and add complexity as needed" |

---

## Final Tips

1. **Practice out loud** - Record yourself explaining designs
2. **Draw diagrams** - Visual communication is powerful
3. **Know your numbers** - Latency, throughput, storage estimates
4. **Learn from feedback** - Each interview is a learning opportunity
5. **Stay calm** - It's okay not to know everything
6. **Be collaborative** - Treat it as a discussion, not a test
