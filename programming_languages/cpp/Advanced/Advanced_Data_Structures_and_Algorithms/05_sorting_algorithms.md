# Sorting Algorithms

Sorting is a fundamental problem in computer science. There are many different sorting algorithms, each with its own advantages and disadvantages.

## Comparison Sorts

Comparison sorts are algorithms that sort by comparing elements. The best possible worst-case time complexity for a comparison sort is O(n log n).

### Bubble Sort

*   **How it works:** Repeatedly steps through the list, compares adjacent elements and swaps them if they are in the wrong order.
*   **Time Complexity:** O(n<sup>2</sup>)
*   **Space Complexity:** O(1)
*   **Notes:** Simple to implement, but very inefficient. Not used in practice.

### Selection Sort

*   **How it works:** Divides the list into a sorted and an unsorted part. Repeatedly finds the smallest element in the unsorted part and swaps it with the first element of the unsorted part.
*   **Time Complexity:** O(n<sup>2</sup>)
*   **Space Complexity:** O(1)
*   **Notes:** Also simple but inefficient.

### Insertion Sort

*   **How it works:** Builds the final sorted array one item at a time. It iterates through the input elements and inserts each element into its correct position in the sorted part of the array.
*   **Time Complexity:** O(n<sup>2</sup>) in the worst case, but O(n) in the best case (for an already sorted list).
*   **Space Complexity:** O(1)
*   **Notes:** Efficient for small lists and for lists that are already partially sorted.

### Merge Sort

*   **How it works:** A divide-and-conquer algorithm. It divides the list into two halves, recursively sorts each half, and then merges the two sorted halves.
*   **Time Complexity:** O(n log n)
*   **Space Complexity:** O(n)
*   **Notes:** A very efficient and stable sort (preserves the relative order of equal elements).

### Quicksort

*   **How it works:** A divide-and-conquer algorithm. It picks a 'pivot' element and partitions the other elements into two sub-arrays, according to whether they are less than or greater than the pivot. The sub-arrays are then sorted recursively.
*   **Time Complexity:** O(n log n) on average, but O(n<sup>2</sup>) in the worst case.
*   **Space Complexity:** O(log n) on average (due to recursion stack).
*   **Notes:** Often faster in practice than other O(n log n) algorithms like Merge Sort, but the worst-case performance can be a problem. The choice of pivot is critical.

### Heapsort

*   **How it works:** Uses a binary heap data structure. It first builds a max-heap from the input data. Then, it repeatedly extracts the maximum element from the heap and puts it at the end of the sorted array.
*   **Time Complexity:** O(n log n)
*   **Space Complexity:** O(1)
*   **Notes:** An efficient in-place sort, but not stable.

## Non-Comparison Sorts

These algorithms sort without comparing elements. They can be faster than O(n log n) but have certain constraints.

### Counting Sort

*   **How it works:** Works by counting the number of objects that have each distinct key value. It is only suitable for sorting integers in a limited range.
*   **Time Complexity:** O(n + k), where k is the range of the input.
*   **Space Complexity:** O(k)

### Radix Sort

*   **How it works:** Sorts integers by processing individual digits. It can process digits of each number starting from the least significant digit (LSD) or the most significant digit (MSD).
*   **Time Complexity:** O(d * (n + b)), where d is the number of digits, n is the number of elements, and b is the base.

## Sorting in C++

The C++ standard library provides `std::sort` (in `<algorithm>`), which is a highly optimized sorting algorithm. It is typically an **Introsort**, which is a hybrid of Quicksort, Heapsort, and Insertion Sort. It starts with Quicksort and switches to Heapsort if the recursion depth becomes too large (to avoid the O(n<sup>2</sup>) worst case), and it uses Insertion Sort for small partitions.
