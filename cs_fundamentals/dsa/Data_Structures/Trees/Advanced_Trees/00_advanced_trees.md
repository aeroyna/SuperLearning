## Introduction to Advanced Tree Structures

Beyond binary search trees, there lies a category of powerful, specialized tree structures designed for handling range-based queries and updates efficiently. For problems that repeatedly ask for the sum, minimum, or maximum of a particular subarray, these advanced trees can provide solutions that are significantly faster than naive approaches.

Two of the most well-known of these structures are **Segment Trees** and **Fenwick Trees (Binary Indexed Trees)**.

### Use Case
Imagine you have an array, and you need to perform two types of operations on it thousands of times:
1.  **Range Query**: Find the sum of elements from index `i` to `j`.
2.  **Point Update**: Change the value of the element at index `i`.

A naive approach would be:
- **Range Query**: Iterate from `i` to `j` and sum the elements. This is O(n).
- **Point Update**: Directly update the array. This is O(1).

If you have many queries, the O(n) query time becomes a bottleneck. A prefix sum array would make queries O(1), but updates would become O(n). Advanced trees offer a compromise, allowing both operations to be completed in **O(log n)** time.

### Common Structures
- **Segment Tree**: A versatile, recursive tree structure. Each node in the segment tree represents an interval or "segment" of the original array. It's powerful but can be more complex to implement.
- **Fenwick Tree (Binary Indexed Tree or BIT)**: A more specialized and often simpler-to-code structure that excels at prefix-based queries (e.g., "give me the sum of the first `i` elements"). It cleverly uses the bitwise representation of indices to achieve its efficiency.

While less common in generalist interviews than core data structures, they frequently appear in competitive programming and interviews for roles requiring strong algorithm skills.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)