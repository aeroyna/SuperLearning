# Practice Problems: Stacks and Queues

Master LIFO/FIFO patterns, monotonic stacks, and parentheses problems.

## Basic Stack

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | Easy | Push openers, pop and match closers. |
| [Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Store `(val, current_min)` tuple or use auxiliary stack. |
| [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium | Push operands, pop 2 on operator, push result. |

## Monotonic Stack

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Medium | Decreasing stack stores indices. Resolve when current > stack top. |
| [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | Easy | Monotonic stack + Hash Map. |
| [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard | Increasing stack. When `h[i] < top`, compute area for popped bar. |

## Queues & BFS

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | Easy | Two stacks: `input` and `output`. Amortized O(1). |
| [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | Monotonic Deque (decreasing). Store indices. |
