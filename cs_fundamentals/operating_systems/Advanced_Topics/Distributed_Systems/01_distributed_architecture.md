# Distributed System Architecture\n\n## Distributed System Architecture

### Overview

Client-server, peer-to-peer, microservices

### Architecture

```c
// Distributed system communication
typedef struct {
    int node_id;
    char *message;
    timestamp_t timestamp;
} DistributedMessage;

void send_message(int dest_node, DistributedMessage *msg) {
    // Network communication
    // Handle failures, retries
}
```

### Implementation

**Python RPC**:
```python
from xmlrpc.server import SimpleXMLRPCServer

def remote_function(arg):
    return f"Result: {arg}"

server = SimpleXMLRPCServer(("localhost", 8000))
server.register_function(remote_function)
server.serve_forever()
```

**Java RMI**:
```java
import java.rmi.*;

public interface RemoteInterface extends Remote {
    String remoteMethod(String arg) throws RemoteException;
}

// Client
RemoteInterface stub = (RemoteInterface) registry.lookup("Service");
String result = stub.remoteMethod("data");
```

### Challenges

- **Consistency**: Maintaining coherent state
- **Availability**: Handling node failures
- **Partition tolerance**: Network splits
- **Latency**: Communication delays

## Key Takeaways

1. Distributed systems span multiple nodes
2. CAP theorem: Choose 2 of consistency, availability, partition tolerance
3. Coordination requires distributed algorithms
4. Fault tolerance is essential

## Interview Focus

1. Explain distributed system challenges
2. What is CAP theorem?
3. How does consensus work (Paxos/Raft)?
4. Compare distributed file systems
