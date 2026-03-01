# Bit Manipulation Tricks

Bit manipulation enables elegant solutions for certain problems with O(1) space and O(n) or O(1) time.

## Essential Bit Operations

| Operation | Code | Description |
|-----------|------|-------------|
| Set bit | `x \|= (1 << n)` | Set nth bit to 1 |
| Clear bit | `x &= ~(1 << n)` | Set nth bit to 0 |
| Toggle bit | `x ^= (1 << n)` | Flip nth bit |
| Check bit | `(x >> n) & 1` | Get nth bit |
| Clear lowest set bit | `x & (x - 1)` | Turn off rightmost 1 |
| Get lowest set bit | `x & (-x)` | Isolate rightmost 1 |
| Check power of 2 | `x && !(x & (x-1))` | True if exactly one bit set |

## XOR Properties

- `a ^ a = 0` (same numbers cancel)
- `a ^ 0 = a` (identity)
- `a ^ b ^ a = b` (self-inverse)
- Commutative and associative

## Classic Problems

### Single Number (One Unique)

```cpp
int singleNumber(std::vector<int>& nums) {
    int result = 0;
    for (int num : nums) {
        result ^= num;
    }
    return result;
}
// Every number appears twice except one
// a ^ a = 0, so pairs cancel out
```

### Single Number II (All Others Appear 3 Times)

```cpp
int singleNumber(std::vector<int>& nums) {
    int ones = 0, twos = 0;
    for (int num : nums) {
        ones = (ones ^ num) & ~twos;
        twos = (twos ^ num) & ~ones;
    }
    return ones;
}
```

### Two Single Numbers

```cpp
std::vector<int> singleNumber(std::vector<int>& nums) {
    int xorAll = 0;
    for (int num : nums) xorAll ^= num;

    // Get rightmost set bit (differs between two numbers)
    int diff = xorAll & (-xorAll);

    int a = 0, b = 0;
    for (int num : nums) {
        if (num & diff) {
            a ^= num;
        } else {
            b ^= num;
        }
    }

    return {a, b};
}
```

### Count Set Bits (Brian Kernighan)

```cpp
int countBits(int n) {
    int count = 0;
    while (n) {
        n &= (n - 1);  // Clear lowest set bit
        ++count;
    }
    return count;
}
// __builtin_popcount(n) in GCC
```

### Power of Two

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

### Reverse Bits

```cpp
uint32_t reverseBits(uint32_t n) {
    uint32_t result = 0;
    for (int i = 0; i < 32; ++i) {
        result = (result << 1) | (n & 1);
        n >>= 1;
    }
    return result;
}
```

### Missing Number (0 to n, one missing)

```cpp
int missingNumber(std::vector<int>& nums) {
    int xorAll = nums.size();  // XOR with n
    for (int i = 0; i < nums.size(); ++i) {
        xorAll ^= i ^ nums[i];
    }
    return xorAll;
}
```

### Generate Subsets Using Bits

```cpp
std::vector<std::vector<int>> subsets(std::vector<int>& nums) {
    int n = nums.size();
    std::vector<std::vector<int>> result;

    for (int mask = 0; mask < (1 << n); ++mask) {
        std::vector<int> subset;
        for (int i = 0; i < n; ++i) {
            if (mask & (1 << i)) {
                subset.push_back(nums[i]);
            }
        }
        result.push_back(subset);
    }

    return result;
}
```

### Hamming Distance

```cpp
int hammingDistance(int x, int y) {
    int diff = x ^ y;  // Bits that differ
    return __builtin_popcount(diff);
}
```

### Swap Without Temp

```cpp
void swap(int& a, int& b) {
    a ^= b;
    b ^= a;
    a ^= b;
}
// But std::swap is clearer and equally efficient
```

## Useful Patterns

### Iterate Through Set Bits

```cpp
while (mask) {
    int lowestBit = mask & (-mask);  // Get lowest set bit
    // Use lowestBit...
    mask &= (mask - 1);  // Clear it
}
```

### All Subsets of a Bitmask

```cpp
// Enumerate all submasks of mask
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // Process sub
}
```

## GCC Built-ins

| Function | Description |
|----------|-------------|
| `__builtin_popcount(x)` | Count set bits |
| `__builtin_clz(x)` | Leading zeros |
| `__builtin_ctz(x)` | Trailing zeros |
| `__builtin_ffs(x)` | Position of first set bit (1-indexed) |

## Key Takeaways

- XOR for finding unique elements
- `n & (n-1)` clears lowest set bit
- `n & (-n)` isolates lowest set bit
- Bitmasks represent subsets
- Bit manipulation = O(1) space tricks
- Use built-ins for counting bits

## Common Interview Questions

> [!question]- Why does `n & (n-1)` clear the lowest set bit?
> `n-1` flips all bits from the lowest set bit to the right. AND with n keeps everything above unchanged and clears from that bit down.

> [!question]- How can XOR find a single unique number?
> XOR of two identical numbers is 0. XOR of any number with 0 is itself. So XORing all elements leaves only the unique one.
