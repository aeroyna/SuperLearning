## Introduction to Sorting

Sorting is the process of arranging a collection of items into a specific order, most commonly ascending or descending. It is one of the most fundamental and heavily studied problems in computer science, and it serves as a foundational building block for a vast number of other algorithms (for example, binary search requires a sorted array).

### What Can Be Sorted?
While we often think of sorting numbers, the concept applies to any data with a defined order:
- **Numbers**: Ascending or descending arithmetic order.
- **Characters/Strings**: Alphabetical (lexicographical) order.
- **Dates/Times**: Chronological order.
- **Custom Objects**: You can sort custom objects (like a `Student` or `Product` class) by defining a **sorting key** (e.g., sort students by GPA, sort products by price).

### Key Features of Sorting Algorithms
When evaluating a sorting algorithm, especially in an interview context, it's important to consider several key properties:

1.  **Time Complexity**: How does the algorithm's runtime scale with the input size (`n`)? This is often the most important factor. An O(n log n) algorithm is significantly better than an O(n^2) algorithm for large inputs.
2.  **Space Complexity (In-place vs. Out-of-place)**:
    -   An **in-place** algorithm requires only a constant amount of extra memory (O(1)). It sorts the elements within the original array by swapping them. (e.g., Heap Sort).
    -   An **out-of-place** algorithm requires extra memory to store the data, often proportional to the input size O(n). (e.g., Merge Sort).
3.  **Stability**: A sorting algorithm is **stable** if it preserves the original relative order of equal elements. For example, if you sort a list of `(name, score)` tuples by score, a stable sort will ensure that if two people have the same score, their original order is maintained. This can be an important property in certain applications.

### Categories of Sorting Algorithms
- **Comparison Sorts**: These algorithms sort elements by comparing them to each other. Their time complexity is provably lower-bounded by O(n log n). This category includes Merge Sort, Quick Sort, and Heap Sort.
- **Non-Comparison Sorts**: These algorithms do not rely on comparisons. They use other properties of the data (like their integer values) to sort them, which can allow them to achieve linear time complexity O(n) in certain scenarios. This category includes Radix Sort and Counting Sort.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)