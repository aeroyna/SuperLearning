# Stack and Heap

Understanding the distinction between the stack and the heap is vital for predicting variable lifetime and performance. The stack is a region of fast, contiguous memory used for static memory allocation (function calls, local variables), while the heap is a larger pool of memory used for dynamic allocation. This chapter dives into how these regions function, their size limits, and the performance implications of allocating on one versus the other.

You will learn about:
- **Stack Allocation:** Automatic storage duration, stack frames, and speed.
- **Heap Allocation:** Dynamic storage duration, fragmentation, and manual management costs.
- **Stack Overflow:** What happens when the stack limit is exceeded.

## In this chapter

- **[Stack vs Heap](01_stack_vs_heap.md)**
