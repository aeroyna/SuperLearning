# Caching

Caching stores frequently accessed data in a fast storage layer to reduce latency and database load. It's one of the most effective techniques for improving system performance.

---

## Why Caching?

```
Without Cache:
Client → Server → Database (100ms)

With Cache:
Client → Server → Cache (1ms) ← Hit!
Client → Server → Database → Cache (100ms) ← Miss, then cache
```

### Benefits
- **Reduced latency**: Memory access is orders of magnitude faster than disk
- **Reduced database load**: Fewer queries hit the database
- **Improved throughput**: Serve more requests with same infrastructure
- **Cost savings**: Less database scaling needed

---

## Topics in This Section

- [4.1 Caching Strategies](01_caching_strategies.md)
- [4.2 Cache Eviction Policies](02_cache_eviction_policies.md)
- [4.3 Distributed Caching](03_distributed_caching.md)

---

## Cache Locations

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client                                   │
│  ┌─────────────┐                                                │
│  │Browser Cache│ ← Closest to user, fastest                     │
│  └─────────────┘                                                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                          CDN                                     │
│  ┌─────────────┐                                                │
│  │ Edge Cache  │ ← Static assets, geographically distributed    │
│  └─────────────┘                                                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     Application Layer                            │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ Local Cache │  │Redis/Memcached│ ← Session, hot data         │
│  │ (in-memory) │  │(distributed) │                              │
│  └─────────────┘  └─────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      Database Layer                              │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │Query Cache  │  │Buffer Pool  │ ← Database internal caching   │
│  └─────────────┘  └─────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## Cache Hit Ratio

```
Hit Ratio = Cache Hits / (Cache Hits + Cache Misses)

Good: 95%+ for hot data
Acceptable: 80%+ for general caching
Poor: <60% - reconsider caching strategy
```

### Monitoring

```python
class CacheMetrics:
    def __init__(self):
        self.hits = 0
        self.misses = 0

    def get(self, key):
        value = self.cache.get(key)
        if value:
            self.hits += 1
        else:
            self.misses += 1
        return value

    def hit_ratio(self):
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0
```

---

## Common Caching Technologies

| Technology | Type | Use Case |
|------------|------|----------|
| Redis | In-memory, distributed | Session, caching, pub/sub |
| Memcached | In-memory, distributed | Simple key-value caching |
| Caffeine | In-process (Java) | Application-level cache |
| Guava Cache | In-process (Java) | Application-level cache |
| Varnish | HTTP accelerator | CDN, reverse proxy cache |

---

## Interview Quick Reference

```
When discussing caching:

1. Identify what to cache:
   - Frequently accessed data
   - Expensive computations
   - Data that doesn't change often

2. Choose cache location:
   - Client-side (browser)
   - CDN (static assets)
   - Application (Redis/Memcached)
   - Database (query cache)

3. Determine caching strategy:
   - Cache-aside
   - Write-through
   - Write-behind

4. Handle cache invalidation:
   - TTL-based
   - Event-driven
   - Versioning

5. Consider failure modes:
   - Cache miss storm
   - Cache stampede
   - Hot keys
```
