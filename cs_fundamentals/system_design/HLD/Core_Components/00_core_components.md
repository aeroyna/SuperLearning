# Core Components

These are the building blocks used in almost every large-scale system. Understanding each component's purpose, trade-offs, and when to use it is essential.

---

## Component Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Internet                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CDN (Static Assets)                                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Load Balancer                                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│                             API Gateway                                      │
│            (Authentication, Rate Limiting, Routing)                          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
            │  Service A  │ │  Service B  │ │  Service C  │
            └─────────────┘ └─────────────┘ └─────────────┘
                    │               │               │
                    └───────────────┼───────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ↓               ↓               ↓
            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
            │    Cache    │ │  Database   │ │Message Queue│
            │   (Redis)   │ │(PostgreSQL) │ │  (Kafka)    │
            └─────────────┘ └─────────────┘ └─────────────┘
```

---

## Topics in This Section

- [6.1 Load Balancers](01_load_balancers.md)
- [6.2 Content Delivery Networks (CDN)](02_cdn.md)
- [6.3 Message Queues](03_message_queues.md)
- [6.4 API Gateway](04_api_gateway.md)
- [6.5 Service Discovery](05_service_discovery.md)
- [6.6 Reverse Proxy](06_reverse_proxy.md)

---

## Quick Reference

| Component | Purpose | Examples |
|-----------|---------|----------|
| Load Balancer | Distribute traffic | Nginx, HAProxy, AWS ALB |
| CDN | Cache static content globally | CloudFront, Cloudflare, Akamai |
| Message Queue | Async communication | Kafka, RabbitMQ, SQS |
| API Gateway | Request routing, auth | Kong, AWS API Gateway |
| Service Discovery | Find service instances | Consul, etcd, Kubernetes |
| Reverse Proxy | Forward requests, caching | Nginx, Varnish |

---

## When to Use Each Component

```
Need to distribute traffic?
→ Load Balancer

Static content with global users?
→ CDN

Async processing or decoupling?
→ Message Queue

Cross-cutting concerns (auth, rate limit)?
→ API Gateway

Dynamic service instances?
→ Service Discovery

Caching, SSL termination, compression?
→ Reverse Proxy
```
