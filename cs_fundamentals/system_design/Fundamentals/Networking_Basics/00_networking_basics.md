# Networking Basics

Understanding networking protocols is essential for designing distributed systems. This section covers the communication patterns commonly discussed in system design interviews.

---

## Topics in This Section

- [5.1 HTTP/HTTPS & REST](01_http_rest.md)
- [5.2 WebSockets & Long Polling](02_websockets_long_polling.md)
- [5.3 gRPC & Protocol Buffers](03_grpc_protobuf.md)

---

## Communication Patterns Overview

| Pattern | Use Case | Latency | Complexity |
|---------|----------|---------|------------|
| HTTP REST | CRUD APIs | Medium | Low |
| GraphQL | Flexible queries | Medium | Medium |
| WebSocket | Real-time bidirectional | Low | Medium |
| Server-Sent Events | Server push | Low | Low |
| gRPC | Service-to-service | Very Low | Medium |
| Message Queue | Async processing | Variable | Medium |

---

## Choosing Communication Protocol

```
Need real-time bidirectional?
├── Yes → WebSocket
└── No
    ├── Server push only? → SSE or Long Polling
    └── Request-response
        ├── External API? → REST or GraphQL
        └── Internal services
            ├── High performance needed? → gRPC
            └── Otherwise → REST
```

---

## Quick Reference

### REST API Design
```
GET    /users          - List users
GET    /users/:id      - Get user
POST   /users          - Create user
PUT    /users/:id      - Replace user
PATCH  /users/:id      - Update user
DELETE /users/:id      - Delete user
```

### WebSocket vs REST

```
REST: Request → Response (stateless)
┌────────┐        ┌────────┐
│ Client │───────→│ Server │
│        │←───────│        │
└────────┘        └────────┘

WebSocket: Persistent connection (stateful)
┌────────┐←──────→┌────────┐
│ Client │        │ Server │
│        │←──────→│        │
└────────┘        └────────┘
(bidirectional, full-duplex)
```

### gRPC vs REST

```
REST: Text-based (JSON), HTTP/1.1
- Human readable
- Larger payload
- Higher latency

gRPC: Binary (Protocol Buffers), HTTP/2
- Machine efficient
- Smaller payload
- Lower latency
- Streaming support
```
