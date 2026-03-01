# Pub/Sub and Streams

## Learning Objectives
- Implement real-time messaging with Pub/Sub
- Build event-driven systems with Redis Streams
- Choose between Pub/Sub and Streams for different use cases
- Design reliable message processing patterns

---

## 1. Pub/Sub (Publish/Subscribe)

### How Pub/Sub Works

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Pub/Sub Architecture                              │
│                                                                      │
│                         PUBLISH "news:sports" "Goal!"                │
│                                      │                               │
│                                      ▼                               │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                       Redis Server                           │    │
│  │                                                              │    │
│  │   Channel: "news:sports"                                     │    │
│  │   ┌──────────────────────────────────────────────────────┐  │    │
│  │   │  Subscribers: [Client1, Client2, Client3]            │  │    │
│  │   └──────────────────────────────────────────────────────┘  │    │
│  │                                                              │    │
│  │   Channel: "news:tech"                                       │    │
│  │   ┌──────────────────────────────────────────────────────┐  │    │
│  │   │  Subscribers: [Client2, Client4]                     │  │    │
│  │   └──────────────────────────────────────────────────────┘  │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                    │              │              │                   │
│                    ▼              ▼              ▼                   │
│              ┌─────────┐   ┌─────────┐   ┌─────────┐                │
│              │ Client1 │   │ Client2 │   │ Client3 │                │
│              │ "Goal!" │   │ "Goal!" │   │ "Goal!" │                │
│              └─────────┘   └─────────┘   └─────────┘                │
│                                                                      │
│  Characteristics:                                                    │
│  • Fire-and-forget (no persistence)                                 │
│  • At-most-once delivery                                            │
│  • No message history                                               │
│  • Real-time only                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Basic Commands

```redis
# Subscribe to channels
SUBSCRIBE channel1 channel2 channel3

# Subscribe with pattern
PSUBSCRIBE news:*              # Matches news:sports, news:tech, etc.
PSUBSCRIBE user:*:notifications

# Publish message
PUBLISH channel1 "Hello subscribers!"
# Returns number of subscribers who received it

# Unsubscribe
UNSUBSCRIBE channel1
PUNSUBSCRIBE news:*

# List channels with subscribers
PUBSUB CHANNELS              # All active channels
PUBSUB CHANNELS news:*       # Matching pattern
PUBSUB NUMSUB channel1       # Subscriber count
PUBSUB NUMPAT                # Pattern subscription count
```

### Subscriber Example (Python)

```python
import redis

r = redis.Redis()
pubsub = r.pubsub()

# Subscribe to channels
pubsub.subscribe('notifications')
pubsub.psubscribe('events:*')

# Listen for messages
for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"Channel: {message['channel']}")
        print(f"Data: {message['data']}")
    elif message['type'] == 'pmessage':
        print(f"Pattern: {message['pattern']}")
        print(f"Channel: {message['channel']}")
        print(f"Data: {message['data']}")
```

### Publisher Example (Python)

```python
import redis

r = redis.Redis()

# Simple publish
r.publish('notifications', 'New message!')
r.publish('events:user:login', 'user:123 logged in')

# With payload
import json
event = {
    'type': 'order_placed',
    'user_id': 123,
    'order_id': 456,
    'amount': 99.99
}
r.publish('events:orders', json.dumps(event))
```

### Pub/Sub Use Cases

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Pub/Sub Use Cases                                 │
│                                                                      │
│  1. Real-time Notifications                                          │
│     ┌─────────┐     ┌─────────┐     ┌─────────┐                     │
│     │ Server  │────▶│  Redis  │────▶│ Clients │                     │
│     │(publish)│     │(channel)│     │(display)│                     │
│     └─────────┘     └─────────┘     └─────────┘                     │
│                                                                      │
│  2. Chat Rooms                                                       │
│     Channel per room: "chat:room:123"                               │
│     Users subscribe when joining, unsubscribe when leaving          │
│                                                                      │
│  3. Live Updates                                                     │
│     Stock prices, sports scores, game events                        │
│     Pattern: "prices:*", "scores:*"                                 │
│                                                                      │
│  4. Invalidation Signals                                             │
│     "cache:invalidate:user:123"                                     │
│     Tell all app servers to refresh cache                           │
│                                                                      │
│  5. Distributed Events                                               │
│     Coordinate between microservices                                │
│     "events:user:created", "events:order:shipped"                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Pub/Sub Limitations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Pub/Sub Limitations                               │
│                                                                      │
│  ✗ No persistence - messages lost if no subscribers                │
│  ✗ No acknowledgment - can't confirm delivery                      │
│  ✗ No replay - can't get historical messages                       │
│  ✗ No consumer groups - all subscribers get all messages           │
│  ✗ Client disconnect = message loss                                 │
│                                                                      │
│  For reliable messaging, use Redis Streams instead                  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Redis Streams

### How Streams Work

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Redis Streams Architecture                        │
│                                                                      │
│  Stream: events                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  1705344000000-0      1705344001000-0      1705344002000-0  │    │
│  │  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐  │    │
│  │  │ type: login   │   │ type: click   │   │ type: purchase│  │    │
│  │  │ user: alice   │   │ user: bob     │   │ user: alice   │  │    │
│  │  │ time: 10:00   │   │ page: /home   │   │ amount: 99.99 │  │    │
│  │  └───────────────┘   └───────────────┘   └───────────────┘  │    │
│  │         │                   │                   │            │    │
│  │         ▼                   ▼                   ▼            │    │
│  │    Entry ID           Entry ID            Entry ID           │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Features:                                                           │
│  • Persistent - entries stay until deleted                          │
│  • Ordered by time (entry ID = timestamp-sequence)                  │
│  • Consumer groups for load balancing                               │
│  • Acknowledgment mechanism                                         │
│  • Replay from any point                                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Basic Stream Operations

```redis
# Add entry to stream
XADD events * type login user alice ip 192.168.1.1
# Returns: 1705344000000-0 (timestamp-sequence)

# Add with specific ID
XADD events 1705344000000-0 type login user alice

# Add with max length (capped stream)
XADD events MAXLEN ~ 1000 * type event data value
# ~ means approximate (more efficient)

# Read entries
XREAD STREAMS events 0              # From beginning
XREAD COUNT 10 STREAMS events 0     # First 10

# Read new entries (blocking)
XREAD BLOCK 5000 STREAMS events $   # Wait 5 sec for new entries

# Range queries
XRANGE events - +                   # All entries
XRANGE events 1705344000000 1705344999999  # Time range
XRANGE events - + COUNT 10          # First 10
XREVRANGE events + - COUNT 10       # Last 10

# Stream info
XLEN events                         # Entry count
XINFO STREAM events                 # Stream metadata
```

### Entry IDs

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Entry ID Format                                   │
│                                                                      │
│  1705344000000-0                                                     │
│  │             │                                                     │
│  │             └── Sequence number (for same millisecond)           │
│  │                                                                   │
│  └──────────────── Millisecond timestamp                            │
│                                                                      │
│  Special IDs:                                                        │
│  *     Auto-generate (XADD)                                         │
│  $     Latest entry (XREAD)                                         │
│  >     Next undelivered (consumer groups)                           │
│  -     Minimum ID (XRANGE)                                          │
│  +     Maximum ID (XRANGE)                                          │
│  0     Start of stream                                              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Consumer Groups

### Consumer Group Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Consumer Groups                                   │
│                                                                      │
│  Stream: orders                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ [Entry1] [Entry2] [Entry3] [Entry4] [Entry5] [Entry6] ...   │    │
│  └───────────────────────────┬─────────────────────────────────┘    │
│                              │                                       │
│          ┌───────────────────┼───────────────────┐                  │
│          ▼                   ▼                   ▼                   │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐          │
│  │ Group: emails │   │ Group: notify │   │ Group: analytics│         │
│  │               │   │               │   │               │          │
│  │ Consumer1 ──┐ │   │ Consumer1     │   │ Consumer1 ──┐ │          │
│  │ Consumer2 ─┬┘ │   │               │   │ Consumer2 ─┬┘ │          │
│  │ Consumer3 ─┘  │   │               │   │ Consumer3 ─┘  │          │
│  └───────────────┘   └───────────────┘   └───────────────┘          │
│                                                                      │
│  • Each group tracks its own position                               │
│  • Entries distributed among consumers in group                     │
│  • Each entry delivered to ONE consumer per group                   │
│  • Different groups get the SAME entries                            │
└─────────────────────────────────────────────────────────────────────┘
```

### Consumer Group Commands

```redis
# Create consumer group
XGROUP CREATE orders order-processors 0 MKSTREAM
# MKSTREAM creates stream if doesn't exist
# 0 means start from beginning
# $ means start from new entries only

# Read as consumer
XREADGROUP GROUP order-processors consumer1 COUNT 10 STREAMS orders >
# > means only new, undelivered entries

# Acknowledge processing
XACK orders order-processors 1705344000000-0

# Check pending entries
XPENDING orders order-processors          # Summary
XPENDING orders order-processors - + 10   # Details

# Claim stuck entries (consumer died)
XCLAIM orders order-processors consumer2 60000 1705344000000-0
# 60000 = min idle time in ms

# Auto-claim (Redis 6.2+)
XAUTOCLAIM orders order-processors consumer2 60000 0-0 COUNT 10

# Delete consumer
XGROUP DELCONSUMER orders order-processors consumer1

# Delete group
XGROUP DESTROY orders order-processors

# Consumer group info
XINFO GROUPS orders
XINFO CONSUMERS orders order-processors
```

### Consumer Implementation (Python)

```python
import redis
import time

r = redis.Redis()
stream = 'orders'
group = 'order-processors'
consumer = 'consumer-1'

# Create group (if not exists)
try:
    r.xgroup_create(stream, group, id='0', mkstream=True)
except redis.ResponseError as e:
    if 'BUSYGROUP' not in str(e):
        raise

# Process messages
while True:
    # Read new entries
    entries = r.xreadgroup(
        group, consumer,
        {stream: '>'},
        count=10,
        block=5000  # 5 second timeout
    )

    if not entries:
        continue

    for stream_name, messages in entries:
        for msg_id, data in messages:
            try:
                # Process message
                print(f"Processing {msg_id}: {data}")
                process_order(data)

                # Acknowledge success
                r.xack(stream, group, msg_id)

            except Exception as e:
                print(f"Error processing {msg_id}: {e}")
                # Don't ACK - will be reprocessed or claimed

# Handle pending (stuck) messages
def recover_pending():
    pending = r.xpending_range(stream, group, '-', '+', 100)
    for entry in pending:
        if entry['time_since_delivered'] > 60000:  # 1 minute
            # Claim and reprocess
            r.xclaim(stream, group, consumer, 60000, entry['message_id'])
```

---

## 4. Stream Patterns

### Event Sourcing

```redis
# Append events to stream
XADD user:123:events * type PROFILE_UPDATED field name value "John Doe"
XADD user:123:events * type EMAIL_CHANGED old john@old.com new john@new.com
XADD user:123:events * type PASSWORD_CHANGED timestamp 1705344000

# Rebuild state by replaying events
XRANGE user:123:events - +

# Snapshot + Events pattern
# 1. Store periodic snapshots in Hash
HSET user:123:snapshot name "John" email "john@new.com" version 1705344000-5

# 2. Replay only events after snapshot
XRANGE user:123:events 1705344000-5 +
```

### Message Queue with Retry

```python
def process_with_retry(stream, group, consumer, max_retries=3):
    while True:
        # First, check pending messages
        pending = r.xpending_range(stream, group, '-', '+', 10,
                                    consumername=consumer)

        for entry in pending:
            msg_id = entry['message_id']
            delivery_count = entry['times_delivered']

            if delivery_count > max_retries:
                # Move to dead letter stream
                msg = r.xrange(stream, msg_id, msg_id)
                if msg:
                    r.xadd('dead_letters', {'original': msg_id, **msg[0][1]})
                    r.xack(stream, group, msg_id)
                continue

            # Retry processing
            try:
                msg = r.xrange(stream, msg_id, msg_id)
                process(msg[0][1])
                r.xack(stream, group, msg_id)
            except:
                pass  # Will retry on next iteration

        # Then process new messages
        entries = r.xreadgroup(group, consumer, {stream: '>'}, count=10)
        # ... process as normal
```

### Fan-Out with Multiple Groups

```redis
# Producer adds to stream once
XADD events * type order_placed order_id 123 user_id 456

# Multiple consumer groups process independently

# Email service group
XGROUP CREATE events email-service 0 MKSTREAM
XREADGROUP GROUP email-service email-worker-1 STREAMS events >

# Analytics group
XGROUP CREATE events analytics-service 0 MKSTREAM
XREADGROUP GROUP analytics-service analytics-worker-1 STREAMS events >

# Inventory group
XGROUP CREATE events inventory-service 0 MKSTREAM
XREADGROUP GROUP inventory-service inventory-worker-1 STREAMS events >
```

### Capped Streams

```redis
# Limit by count (approximate for performance)
XADD logs MAXLEN ~ 10000 * level info message "Something happened"

# Limit by count (exact)
XADD logs MAXLEN 10000 * level info message "Something happened"

# Limit by ID/time (trim entries older than ID)
XTRIM logs MINID ~ 1705344000000-0

# Manual trim
XTRIM logs MAXLEN 5000
```

---

## 5. Pub/Sub vs Streams

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Comparison                                        │
│                                                                      │
│  Feature              │  Pub/Sub          │  Streams                │
│  ─────────────────────┼───────────────────┼─────────────────────────│
│  Persistence          │  No               │  Yes                    │
│  Message History      │  No               │  Yes                    │
│  Consumer Groups      │  No               │  Yes                    │
│  Acknowledgment       │  No               │  Yes                    │
│  Replay               │  No               │  Yes (any point)        │
│  Delivery             │  At-most-once     │  At-least-once          │
│  Load Balancing       │  No (broadcast)   │  Yes (per group)        │
│  Blocking Read        │  Yes              │  Yes                    │
│  Pattern Subscribe    │  Yes              │  No                     │
│  Latency              │  Lower            │  Slightly higher        │
│  Complexity           │  Simple           │  More complex           │
│                                                                      │
│  Use Pub/Sub for:                                                    │
│  • Real-time notifications where loss is OK                         │
│  • Simple broadcast to all subscribers                              │
│  • Cache invalidation signals                                       │
│  • Live updates (sports, stocks)                                    │
│                                                                      │
│  Use Streams for:                                                    │
│  • Reliable message processing                                      │
│  • Event sourcing                                                   │
│  • Task queues with acknowledgment                                  │
│  • Audit logs                                                       │
│  • Fan-out to multiple independent consumers                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Monitoring and Operations

### Stream Monitoring

```redis
# Stream info
XINFO STREAM mystream
# length: 1000
# first-entry: [id, fields]
# last-entry: [id, fields]
# groups: 2

# Group info
XINFO GROUPS mystream
# name, consumers, pending, last-delivered-id

# Consumer info
XINFO CONSUMERS mystream mygroup
# name, pending, idle

# Memory usage
MEMORY USAGE mystream
```

### Maintenance

```redis
# Trim old entries
XTRIM mystream MAXLEN 100000

# Delete specific entries
XDEL mystream 1705344000000-0

# Delete entire stream
DEL mystream

# Reset consumer group position
XGROUP SETID mystream mygroup 0
XGROUP SETID mystream mygroup $  # Skip to end
```

---

## Summary

| Feature | Pub/Sub | Streams |
|---------|---------|---------|
| Model | Broadcast | Log |
| Persistence | No | Yes |
| Acknowledgment | No | Yes |
| Consumer Groups | No | Yes |
| Use Case | Real-time events | Message queue |

---

## Best Practices

```
Pub/Sub:
✓ Use for fire-and-forget real-time events
✓ Handle subscriber disconnects gracefully
✓ Use patterns sparingly (performance impact)
✓ Keep messages small

Streams:
✓ Use consumer groups for reliable processing
✓ Always acknowledge processed messages
✓ Implement dead letter handling for failures
✓ Set MAXLEN to prevent unbounded growth
✓ Monitor pending entries and consumer lag
✓ Use XAUTOCLAIM for stuck message recovery
```
