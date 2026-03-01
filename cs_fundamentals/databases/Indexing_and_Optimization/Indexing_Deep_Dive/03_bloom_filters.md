# Bloom Filters

## Introduction

Bloom filters are space-efficient probabilistic data structures that test whether an element is a member of a set. They can have false positives but never false negatives, making them perfect for reducing unnecessary disk reads in databases.

## Core Concept

```
┌─────────────────────────────────────────────────────────────┐
│                    Bloom Filter Properties                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Query: "Is element X in the set?"                          │
│                                                              │
│  Answer: "DEFINITELY NOT" or "PROBABLY YES"                 │
│                                                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │  False Positive:  Possible (tunable probability)   │     │
│  │  False Negative:  IMPOSSIBLE                       │     │
│  └────────────────────────────────────────────────────┘     │
│                                                              │
│  Use Case:                                                   │
│  "Should I read this SSTable from disk?"                    │
│  If Bloom says NO → Skip (100% correct)                     │
│  If Bloom says YES → Check disk (might be wrong)            │
└─────────────────────────────────────────────────────────────┘
```

## Structure and Operations

### Bit Array Representation

```
┌─────────────────────────────────────────────────────────────┐
│  Bloom Filter: Bit array of m bits, k hash functions        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Initial State (empty):                                      │
│  Position: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15            │
│  Bits:     0 0 0 0 0 0 0 0 0 0  0  0  0  0  0  0            │
│                                                              │
│  k = 3 hash functions: h1, h2, h3                           │
│                                                              │
│  Insert "apple":                                             │
│    h1("apple") mod 16 = 2                                    │
│    h2("apple") mod 16 = 7                                    │
│    h3("apple") mod 16 = 13                                   │
│                                                              │
│  After insert:                                               │
│  Position: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15            │
│  Bits:     0 0 1 0 0 0 0 1 0 0  0  0  0  1  0  0            │
│                ↑           ↑              ↑                  │
│                                                              │
│  Insert "banana":                                            │
│    h1("banana") mod 16 = 5                                   │
│    h2("banana") mod 16 = 7   (collision with "apple")       │
│    h3("banana") mod 16 = 11                                  │
│                                                              │
│  After insert:                                               │
│  Position: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15            │
│  Bits:     0 0 1 0 0 1 0 1 0 0  0  1  0  1  0  0            │
│                ↑     ↑   ↑        ↑      ↑                   │
└─────────────────────────────────────────────────────────────┘
```

### Lookup Operation

```
┌─────────────────────────────────────────────────────────────┐
│  Query: Is "cherry" in the set?                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Current state:                                              │
│  Position: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15            │
│  Bits:     0 0 1 0 0 1 0 1 0 0  0  1  0  1  0  0            │
│                                                              │
│  Calculate hashes:                                           │
│    h1("cherry") mod 16 = 5  → bit[5] = 1 ✓                  │
│    h2("cherry") mod 16 = 9  → bit[9] = 0 ✗                  │
│    h3("cherry") mod 16 = 11 → bit[11] = 1 ✓                 │
│                                                              │
│  Result: At least one bit is 0                               │
│  Answer: DEFINITELY NOT in set                               │
├─────────────────────────────────────────────────────────────┤
│  Query: Is "grape" in the set?                               │
│                                                              │
│  Calculate hashes:                                           │
│    h1("grape") mod 16 = 2  → bit[2] = 1 ✓                   │
│    h2("grape") mod 16 = 5  → bit[5] = 1 ✓                   │
│    h3("grape") mod 16 = 7  → bit[7] = 1 ✓                   │
│                                                              │
│  Result: All bits are 1                                      │
│  Answer: PROBABLY in set (FALSE POSITIVE!)                   │
│  "grape" was never inserted, but matches due to collisions  │
└─────────────────────────────────────────────────────────────┘
```

## Mathematical Analysis

### False Positive Probability

```
┌─────────────────────────────────────────────────────────────┐
│  Parameters:                                                 │
│    m = number of bits in filter                              │
│    n = number of elements inserted                           │
│    k = number of hash functions                              │
│                                                              │
│  Probability of false positive:                              │
│                                                              │
│         ⎛       ⎛     1 ⎞^kn ⎞^k                            │
│    p ≈  ⎜ 1 - e^⎜ - ─── ⎟   ⎟                              │
│         ⎝       ⎝     m ⎠    ⎠                              │
│                                                              │
│  Simplified (for large m):                                   │
│                                                              │
│    p ≈ (1 - e^(-kn/m))^k                                    │
│                                                              │
│  Optimal number of hash functions:                           │
│                                                              │
│    k_optimal = (m/n) × ln(2) ≈ 0.693 × (m/n)               │
│                                                              │
│  At optimal k:                                               │
│                                                              │
│    p ≈ (0.6185)^(m/n)                                       │
└─────────────────────────────────────────────────────────────┘
```

### Sizing Guidelines

```
┌─────────────────────────────────────────────────────────────────┐
│  Bits per Element vs False Positive Rate                        │
├─────────────────────────────────────────────────────────────────┤
│  Bits/Element │ Optimal k │ False Positive Rate                 │
│───────────────│───────────│─────────────────────────────────────│
│       4       │     3     │ 14.7%                               │
│       6       │     4     │ 5.6%                                │
│       8       │     6     │ 2.2%                                │
│      10       │     7     │ 0.82%                               │
│      12       │     8     │ 0.31%                               │
│      14       │    10     │ 0.12%                               │
│      16       │    11     │ 0.046%                              │
│      20       │    14     │ 0.006%                              │
└─────────────────────────────────────────────────────────────────┘

Example Sizing:
┌─────────────────────────────────────────────────────────────┐
│  Requirement: 1% false positive rate                         │
│  Elements: 1,000,000                                         │
│                                                              │
│  Bits needed: n × (-1.44 × log2(p))                         │
│             = 1,000,000 × (-1.44 × log2(0.01))              │
│             = 1,000,000 × 9.58                               │
│             ≈ 9,580,000 bits ≈ 1.2 MB                       │
│                                                              │
│  Optimal k = 0.693 × (9.58) ≈ 7 hash functions              │
└─────────────────────────────────────────────────────────────┘
```

## Implementation

### Basic Implementation

```python
import mmh3  # MurmurHash3
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size: int, num_hashes: int):
        """
        Initialize Bloom filter.

        Args:
            size: Number of bits (m)
            num_hashes: Number of hash functions (k)
        """
        self.size = size
        self.num_hashes = num_hashes
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)

    def _hashes(self, item: str) -> list[int]:
        """Generate k hash values using double hashing."""
        # Use two independent hashes to generate k hashes
        h1 = mmh3.hash(item, 0) % self.size
        h2 = mmh3.hash(item, 1) % self.size

        return [(h1 + i * h2) % self.size
                for i in range(self.num_hashes)]

    def add(self, item: str) -> None:
        """Add item to the filter."""
        for position in self._hashes(item):
            self.bit_array[position] = 1

    def contains(self, item: str) -> bool:
        """Check if item might be in the filter."""
        return all(self.bit_array[pos]
                   for pos in self._hashes(item))

    @classmethod
    def optimal(cls, n: int, p: float) -> 'BloomFilter':
        """Create optimally sized Bloom filter."""
        import math
        m = int(-n * math.log(p) / (math.log(2) ** 2))
        k = int((m / n) * math.log(2))
        return cls(m, max(1, k))


# Usage
bf = BloomFilter.optimal(n=1000000, p=0.01)
bf.add("user:123")
bf.add("user:456")

print(bf.contains("user:123"))  # True
print(bf.contains("user:789"))  # False (probably)
```

### Optimized Hash Generation

```python
def enhanced_double_hashing(item: bytes, k: int, m: int) -> list[int]:
    """
    Enhanced double hashing for Bloom filters.
    Based on Kirsch-Mitzenmacher optimization.

    Uses only 2 hash computations for k hash values.
    """
    # Get 128-bit hash and split into two 64-bit values
    hash128 = mmh3.hash128(item)
    h1 = hash128 & 0xFFFFFFFFFFFFFFFF
    h2 = hash128 >> 64

    positions = []
    for i in range(k):
        # g(i) = h1 + i*h2 + i^2  (enhanced formula)
        combined = (h1 + i * h2 + (i * i * i) // 6) % m
        positions.append(combined)

    return positions
```

## Bloom Filter Variants

### Counting Bloom Filter

```
┌─────────────────────────────────────────────────────────────┐
│                  Counting Bloom Filter                       │
├─────────────────────────────────────────────────────────────┤
│  Instead of bits, use counters (typically 4 bits each)      │
│  Allows DELETE operations                                    │
│                                                              │
│  Structure (4-bit counters):                                 │
│  Position: 0  1  2  3  4  5  6  7  8  9                     │
│  Count:    0  0  2  0  0  1  0  3  0  0                     │
│                 ↑           ↑     ↑                          │
│                 Added 2x    Added once  Added 3x            │
│                                                              │
│  Insert: increment counters at hash positions               │
│  Delete: decrement counters at hash positions               │
│  Lookup: check if all counters > 0                          │
│                                                              │
│  Trade-off: 4x space vs standard Bloom filter               │
│  Risk: Counter overflow (saturate at max value)             │
└─────────────────────────────────────────────────────────────┘
```

```python
class CountingBloomFilter:
    def __init__(self, size: int, num_hashes: int, counter_bits: int = 4):
        self.size = size
        self.num_hashes = num_hashes
        self.max_count = (1 << counter_bits) - 1
        self.counters = [0] * size

    def add(self, item: str) -> None:
        for pos in self._hashes(item):
            if self.counters[pos] < self.max_count:
                self.counters[pos] += 1

    def remove(self, item: str) -> bool:
        positions = self._hashes(item)
        if all(self.counters[pos] > 0 for pos in positions):
            for pos in positions:
                self.counters[pos] -= 1
            return True
        return False

    def contains(self, item: str) -> bool:
        return all(self.counters[pos] > 0
                   for pos in self._hashes(item))
```

### Scalable Bloom Filter

```
┌─────────────────────────────────────────────────────────────┐
│                  Scalable Bloom Filter                       │
├─────────────────────────────────────────────────────────────┤
│  Problem: Standard Bloom filter has fixed capacity          │
│  Solution: Chain of Bloom filters with decreasing FPR       │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ BF₀: n₀ elements, FPR = p × r⁰                      │    │
│  └─────────────────────────────────────────────────────┘    │
│            ↓ Full                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ BF₁: n₁ elements, FPR = p × r¹ (tighter)           │    │
│  └─────────────────────────────────────────────────────┘    │
│            ↓ Full                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ BF₂: n₂ elements, FPR = p × r² (even tighter)      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Total FPR: p × (1 + r + r² + ...) = p / (1 - r)           │
│  Common: r = 0.5, so total FPR ≈ 2p                        │
│                                                              │
│  Lookup: Check ALL filters (any positive = positive)        │
│  Insert: Only to newest filter                               │
└─────────────────────────────────────────────────────────────┘
```

### Cuckoo Filter

```
┌─────────────────────────────────────────────────────────────┐
│                      Cuckoo Filter                           │
├─────────────────────────────────────────────────────────────┤
│  Advantages over Bloom:                                      │
│    ✓ Supports deletion                                       │
│    ✓ Better space efficiency for same FPR                   │
│    ✓ Better lookup performance                               │
│                                                              │
│  Structure: Array of buckets, each holds b fingerprints     │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Bucket 0: [fp1|fp2|fp3|fp4]                            │ │
│  │ Bucket 1: [   |   |fp5|   ]                            │ │
│  │ Bucket 2: [fp6|fp7|   |   ]                            │ │
│  │ ...                                                     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Insert algorithm:                                           │
│  1. Calculate fingerprint: f = fingerprint(x)               │
│  2. Calculate buckets: i₁ = hash(x), i₂ = i₁ ⊕ hash(f)    │
│  3. If either bucket has space, insert fingerprint          │
│  4. Else, evict random entry and relocate (cuckoo style)   │
│                                                              │
│  Lookup: Check both buckets for fingerprint                 │
│  Delete: Remove fingerprint from bucket                      │
└─────────────────────────────────────────────────────────────┘
```

## Database Applications

### SSTable Bloom Filters (LSM Trees)

```
┌─────────────────────────────────────────────────────────────┐
│              Bloom Filter in SSTable Lookup                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Read Request: Get key "user:12345"                         │
│                                                              │
│  Without Bloom Filter:                                       │
│    L0: Read 4 SSTables (all might contain key)              │
│    L1: Read 1 SSTable (binary search by key range)          │
│    L2: Read 1 SSTable                                        │
│    Total: 6 disk reads (worst case)                          │
│                                                              │
│  With Bloom Filter (1% FPR per SSTable):                    │
│    L0 SST1: Bloom says NO  → Skip                           │
│    L0 SST2: Bloom says NO  → Skip                           │
│    L0 SST3: Bloom says YES → Read (false positive)          │
│    L0 SST4: Bloom says YES → Read (found!)                  │
│                                                              │
│    Expected reads: 4 × 0.01 + 1 = 1.04 SSTables             │
│                                                              │
│  Bloom Filter Storage:                                       │
│    10 bits/key × 1M keys = 1.25 MB per SSTable              │
│    Kept in memory for fast access                            │
└─────────────────────────────────────────────────────────────┘
```

### PostgreSQL Bloom Index

```sql
-- Create bloom index extension
CREATE EXTENSION bloom;

-- Create bloom index on multiple columns
CREATE INDEX idx_bloom ON products
USING bloom (category, price, brand, color)
WITH (length = 80, col1 = 2, col2 = 2, col3 = 2, col4 = 2);

-- Query using bloom index
SELECT * FROM products
WHERE category = 'electronics'
  AND price = 299.99
  AND brand = 'acme';

-- Bloom index advantages:
-- • Much smaller than btree for multi-column indexes
-- • Good for equality checks on multiple columns
-- • All columns equally usable (no leftmost prefix rule)

-- Limitations:
-- • No range queries
-- • False positives require heap recheck
-- • Not for unique constraints
```

### Cassandra Bloom Filters

```yaml
# cassandra.yaml configuration
bloom_filter_fp_chance: 0.01  # 1% false positive rate

# Per-table configuration
CREATE TABLE users (
    id uuid PRIMARY KEY,
    name text,
    email text
) WITH bloom_filter_fp_chance = 0.001;  # 0.1% for hot table

# Monitoring bloom filter effectiveness
nodetool cfstats keyspace.table

# Output includes:
# Bloom filter false positives: 12345
# Bloom filter false ratio: 0.00087
# Bloom filter space used: 1.5 MB
```

### Redis Bloom Module

```redis
# Load bloom module
MODULE LOAD /path/to/redisbloom.so

# Create bloom filter with capacity and error rate
BF.RESERVE user_visits 0.001 1000000

# Add elements
BF.ADD user_visits "user:123"
BF.MADD user_visits "user:456" "user:789" "user:012"

# Check membership
BF.EXISTS user_visits "user:123"  # Returns 1
BF.EXISTS user_visits "user:999"  # Returns 0 (probably)

# Get filter info
BF.INFO user_visits
# 1) Capacity
# 2) Size
# 3) Number of filters
# 4) Number of items inserted
# 5) Expansion rate
```

## Performance Optimization

### Memory Layout

```
┌─────────────────────────────────────────────────────────────┐
│              Cache-Friendly Bloom Filter                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Problem: Random bit accesses cause cache misses            │
│                                                              │
│  Solution 1: Block-Based Bloom Filter                       │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Block 0 (512 bits = 1 cache line)                      │ │
│  │ Block 1 (512 bits)                                      │ │
│  │ Block 2 (512 bits)                                      │ │
│  │ ...                                                     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Algorithm:                                                  │
│  1. Hash item to select block                               │
│  2. All k bits set within that block                        │
│  3. Only ONE cache miss per lookup                          │
│                                                              │
│  Solution 2: Partitioned Bloom Filter                       │
│  ┌──────────┬──────────┬──────────┐                        │
│  │ Part 1   │ Part 2   │ Part 3   │                        │
│  │ (h1 only)│ (h2 only)│ (h3 only)│                        │
│  └──────────┴──────────┴──────────┘                        │
│                                                              │
│  Each partition uses different hash                          │
│  Better for parallel hash computation                        │
└─────────────────────────────────────────────────────────────┘
```

### SIMD Optimization

```cpp
// AVX2-optimized bloom filter lookup
#include <immintrin.h>

bool bloom_contains_simd(const uint8_t* filter,
                          uint64_t h1, uint64_t h2,
                          int k, int m) {
    __m256i result = _mm256_set1_epi32(0xFFFFFFFF);

    for (int i = 0; i < k; i += 8) {
        // Calculate 8 bit positions at once
        __m256i indices = calculate_indices_avx2(h1, h2, i, m);

        // Gather bits from filter
        __m256i bits = gather_bits_avx2(filter, indices);

        // AND with running result
        result = _mm256_and_si256(result, bits);
    }

    // Check if all bits were set
    return _mm256_movemask_epi8(result) == 0xFFFFFFFF;
}
```

## Key Takeaways

1. **No false negatives** - If Bloom says NO, it's definitely NO
2. **Tunable false positive rate** - Trade space for accuracy
3. **10 bits/element** gives ~1% FPR with 7 hash functions
4. **Essential for LSM trees** - Dramatically reduce read amplification
5. **Cannot delete from standard Bloom** - Use counting or cuckoo variant
6. **Memory access patterns matter** - Block-based layout for cache efficiency
