# gRPC & Protocol Buffers

gRPC is a high-performance RPC framework using Protocol Buffers for serialization. Widely used for service-to-service communication.

---

## Why gRPC?

### REST vs gRPC

| Aspect | REST | gRPC |
|--------|------|------|
| Protocol | HTTP/1.1 (typically) | HTTP/2 |
| Payload | JSON (text) | Protocol Buffers (binary) |
| Contract | OpenAPI/Swagger | .proto files |
| Streaming | Limited | Native support |
| Code generation | Optional | Built-in |
| Browser support | Native | Requires proxy |

### Performance Benefits

```
JSON: {"userId": 123, "name": "John", "email": "john@example.com"}
Size: 56 bytes

Protocol Buffers: Binary encoded
Size: ~25 bytes (55% smaller)

Plus: Faster serialization/deserialization
```

---

## Protocol Buffers

### Defining a Schema (.proto)

```protobuf
syntax = "proto3";

package user;

// Service definition
service UserService {
    // Unary RPC
    rpc GetUser(GetUserRequest) returns (User);

    // Server streaming
    rpc ListUsers(ListUsersRequest) returns (stream User);

    // Client streaming
    rpc UploadUsers(stream User) returns (UploadResult);

    // Bidirectional streaming
    rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

// Message definitions
message User {
    int64 id = 1;
    string name = 2;
    string email = 3;
    repeated string roles = 4;
    Address address = 5;
    google.protobuf.Timestamp created_at = 6;
}

message Address {
    string street = 1;
    string city = 2;
    string country = 3;
}

message GetUserRequest {
    int64 user_id = 1;
}

message ListUsersRequest {
    int32 page_size = 1;
    string page_token = 2;
}

message UploadResult {
    int32 uploaded_count = 1;
}

message ChatMessage {
    string sender_id = 1;
    string content = 2;
}
```

### Field Numbers

```protobuf
message User {
    int64 id = 1;      // Field number 1 (1-15 use 1 byte)
    string name = 2;   // Field number 2
    string email = 3;  // Field number 3

    // Field numbers 16+ use 2 bytes
    // Reserve 1-15 for frequently used fields
}
```

### Types

```protobuf
// Scalar types
int32, int64, uint32, uint64
float, double
bool
string
bytes

// Complex types
message NestedMessage { ... }
enum Status { UNKNOWN = 0; ACTIVE = 1; INACTIVE = 2; }
repeated string tags = 1;     // List
map<string, int32> scores = 2; // Map
oneof result { Success success = 1; Error error = 2; }  // Union
```

---

## gRPC Communication Patterns

### 1. Unary RPC

Single request, single response (like REST).

```
Client ─── Request ───→ Server
Client ←── Response ─── Server
```

```python
# Client
response = stub.GetUser(GetUserRequest(user_id=123))
print(response.name)

# Server
class UserService(user_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        user = db.get_user(request.user_id)
        return User(id=user.id, name=user.name, email=user.email)
```

### 2. Server Streaming

Single request, stream of responses.

```
Client ─── Request ────────────→ Server
Client ←── Response 1 ────────── Server
Client ←── Response 2 ────────── Server
Client ←── Response N ────────── Server
```

```python
# Client
for user in stub.ListUsers(ListUsersRequest(page_size=100)):
    print(user.name)

# Server
def ListUsers(self, request, context):
    for user in db.get_all_users():
        yield User(id=user.id, name=user.name)
```

**Use case**: Downloading large datasets, live updates

### 3. Client Streaming

Stream of requests, single response.

```
Client ─── Request 1 ──────────→ Server
Client ─── Request 2 ──────────→ Server
Client ─── Request N ──────────→ Server
Client ←── Response ───────────── Server
```

```python
# Client
def generate_users():
    for user_data in large_dataset:
        yield User(name=user_data['name'], email=user_data['email'])

response = stub.UploadUsers(generate_users())
print(f"Uploaded {response.uploaded_count} users")

# Server
def UploadUsers(self, request_iterator, context):
    count = 0
    for user in request_iterator:
        db.insert_user(user)
        count += 1
    return UploadResult(uploaded_count=count)
```

**Use case**: Uploading files, batch operations

### 4. Bidirectional Streaming

Both sides stream simultaneously.

```
Client ─── Request 1 ──────────→ Server
Client ←── Response 1 ─────────── Server
Client ─── Request 2 ──────────→ Server
Client ←── Response 2 ─────────── Server
        (interleaved, full-duplex)
```

```python
# Client
def chat_messages():
    while True:
        message = input("Enter message: ")
        yield ChatMessage(sender_id="user1", content=message)

for response in stub.Chat(chat_messages()):
    print(f"Received: {response.content}")

# Server
def Chat(self, request_iterator, context):
    for message in request_iterator:
        # Process and respond
        yield ChatMessage(sender_id="bot", content=f"Echo: {message.content}")
```

**Use case**: Real-time chat, gaming, live collaboration

---

## Implementation

### Server (Python)

```python
import grpc
from concurrent import futures
import user_pb2
import user_pb2_grpc

class UserServicer(user_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        # Fetch from database
        user = database.get_user(request.user_id)
        if not user:
            context.set_code(grpc.StatusCode.NOT_FOUND)
            context.set_details('User not found')
            return user_pb2.User()

        return user_pb2.User(
            id=user.id,
            name=user.name,
            email=user.email
        )

    def ListUsers(self, request, context):
        users = database.list_users(limit=request.page_size)
        for user in users:
            yield user_pb2.User(id=user.id, name=user.name)

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServiceServicer_to_server(UserServicer(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```

### Client (Python)

```python
import grpc
import user_pb2
import user_pb2_grpc

def run():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = user_pb2_grpc.UserServiceStub(channel)

        # Unary call
        try:
            response = stub.GetUser(user_pb2.GetUserRequest(user_id=123))
            print(f"User: {response.name}")
        except grpc.RpcError as e:
            print(f"Error: {e.code()} - {e.details()}")

        # Streaming call
        for user in stub.ListUsers(user_pb2.ListUsersRequest(page_size=10)):
            print(f"User: {user.name}")

if __name__ == '__main__':
    run()
```

---

## Error Handling

### gRPC Status Codes

```
OK = 0                 # Success
CANCELLED = 1          # Client cancelled
UNKNOWN = 2            # Unknown error
INVALID_ARGUMENT = 3   # Bad request
DEADLINE_EXCEEDED = 4  # Timeout
NOT_FOUND = 5          # Resource not found
ALREADY_EXISTS = 6     # Resource already exists
PERMISSION_DENIED = 7  # No permission
UNAUTHENTICATED = 16   # Not authenticated
RESOURCE_EXHAUSTED = 8 # Rate limited
INTERNAL = 13          # Internal server error
UNAVAILABLE = 14       # Service unavailable
```

```python
def GetUser(self, request, context):
    if request.user_id <= 0:
        context.abort(grpc.StatusCode.INVALID_ARGUMENT, 'Invalid user ID')

    user = db.get_user(request.user_id)
    if not user:
        context.abort(grpc.StatusCode.NOT_FOUND, 'User not found')

    return User(id=user.id, name=user.name)
```

---

## Load Balancing

### Client-Side Load Balancing

```python
# gRPC has built-in load balancing
channel = grpc.insecure_channel(
    'dns:///my-service:50051',
    options=[
        ('grpc.lb_policy_name', 'round_robin'),
    ]
)
```

### Server-Side (with Proxy)

```
Client → Envoy/NGINX → gRPC Servers
```

---

## Deadlines & Timeouts

```python
# Client-side deadline
from datetime import datetime, timedelta

deadline = datetime.now() + timedelta(seconds=5)
response = stub.GetUser(
    request,
    timeout=5.0  # 5 seconds
)

# Server-side check
def GetUser(self, request, context):
    if context.time_remaining() < 0:
        context.abort(grpc.StatusCode.DEADLINE_EXCEEDED, 'Deadline exceeded')
```

---

## When to Use gRPC

```
Use gRPC:
✓ Service-to-service communication
✓ High-performance requirements
✓ Streaming needed
✓ Polyglot environment (many languages)
✓ Strong typing desired

Use REST:
✓ Public APIs (browser clients)
✓ Simple CRUD operations
✓ Human readability important
✓ Broad compatibility needed
```

---

## Interview Tips

1. **Know the trade-offs**: Binary vs text, HTTP/2 vs HTTP/1.1
2. **Streaming types**: Understand all four patterns
3. **Error handling**: gRPC status codes vs HTTP status codes
4. **Schema evolution**: Proto3 backward compatibility
5. **When to use**: Internal services, high performance needs
