# Arrays and Strings

Arrays and strings are the most fundamental data structures and appear in virtually every coding interview. Mastering array manipulation techniques is essential for success.

## Overview

Arrays store elements in contiguous memory locations, enabling O(1) random access. Strings in most languages are immutable arrays of characters with special operations.

## Topics

- [3.1 Array Fundamentals](01_array_fundamentals.md) - Basic operations and patterns
- [3.2 String Manipulation](02_string_manipulation.md) - Common string operations
- [3.3 Two Pointers](Two_Pointers/01_two_pointers.md) - Efficient array traversal
- [3.4 Sliding Window](Sliding_Window/01_sliding_window.md) - Subarray problems
- [3.5 Prefix Sum](Prefix_Sum/01_prefix_sum.md) - Range sum queries
- [3.6 Practice Problems](Practice_Problems/00_practice_problems.md)

## Key Complexity

| Operation | Array | String (Immutable) |
|-----------|-------|-------------------|
| Access by index | O(1) | O(1) |
| Search | O(n) | O(n) |
| Insert at end | O(1)* | O(n) |
| Insert at arbitrary | O(n) | O(n) |
| Delete | O(n) | O(n) |
| Concatenation | N/A | O(n + m) |

*Amortized for dynamic arrays

## Common Interview Patterns

1. **Two Pointers** - Two indices moving through array
   - From both ends toward middle
   - Both from start, different speeds
   - Two arrays simultaneously

2. **Sliding Window** - Variable or fixed-size subarray
   - Find longest/shortest subarray with constraint
   - Count subarrays meeting criteria

3. **Prefix Sum** - Precompute cumulative sums
   - Range sum queries
   - Subarray sum problems

4. **In-place Modification** - Modify array without extra space
   - Remove duplicates
   - Move zeros
   - Reverse operations

5. **Sorting + Additional Logic**
   - Two Sum variants
   - Three Sum
   - Meeting point problems

## Quick Tips

1. **Consider edge cases**: Empty array, single element, all same elements
2. **Watch for off-by-one errors**: Array indices, loop bounds
3. **String immutability**: Use StringBuilder/list for modifications
4. **Integer overflow**: Sum of large arrays may overflow
5. **In-place vs extra space**: Know when each is appropriate
