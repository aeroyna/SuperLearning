# Problem-Solving Framework for Technical Interviews

A systematic approach to tackle any DSA problem in interviews.

## The UMPIRE Method

### U - Understand the Problem

1. **Repeat the problem** back to the interviewer
2. **Ask clarifying questions**:
   - Input constraints (size, range, edge cases)
   - Expected output format
   - Can input be modified?
   - Time/space requirements
3. **Walk through examples** to confirm understanding

### M - Match to Known Patterns

Identify which pattern/technique applies:

| If you see... | Consider... |
|---------------|-------------|
| Sorted array | Binary search |
| Subarray/substring | Sliding window, two pointers |
| Tree/graph | BFS/DFS |
| Optimization | DP, greedy |
| Permutation/combination | Backtracking |
| Top/bottom K | Heap |
| Frequency counting | Hash map |

### P - Plan the Approach

1. **Start with brute force** - Explain it even if not optimal
2. **Identify bottlenecks** - What makes brute force slow?
3. **Optimize** - Apply appropriate technique
4. **Write pseudocode** if helpful
5. **Confirm approach** with interviewer before coding

### I - Implement

1. **Write clean, readable code**
2. **Use meaningful variable names**
3. **Handle edge cases**
4. **Talk through your code** as you write

### R - Review

1. **Trace through with an example**
2. **Check edge cases**: empty input, single element, duplicates
3. **Verify logic at boundaries**

### E - Evaluate

1. **Time complexity** - Explain your analysis
2. **Space complexity** - Include recursion stack
3. **Can it be optimized further?**

## Common Patterns Quick Reference

### Array/String Patterns

| Pattern | Time | When to Use |
|---------|------|-------------|
| Two Pointers | O(n) | Sorted arrays, palindromes |
| Sliding Window | O(n) | Subarray constraints |
| Prefix Sum | O(n) + O(1) | Range sum queries |
| Binary Search | O(log n) | Sorted, monotonic |

### Tree/Graph Patterns

| Pattern | When to Use |
|---------|-------------|
| DFS | Path finding, connected components |
| BFS | Shortest path (unweighted), level order |
| Topological Sort | Dependencies, ordering |

### Optimization Patterns

| Pattern | When to Use |
|---------|-------------|
| Dynamic Programming | Overlapping subproblems, optimal value |
| Greedy | Local choice → global optimal |
| Backtracking | Explore all possibilities |

## Communication Tips

### Do's

- Think out loud
- Explain your reasoning
- Ask questions when stuck
- Acknowledge trade-offs
- Test your solution verbally

### Don'ts

- Don't code in silence
- Don't give up without trying
- Don't argue with hints
- Don't ignore edge cases

## Handling Edge Cases

Always consider:

1. **Empty input**: `[]`, `""`
2. **Single element**: `[1]`, `"a"`
3. **All same elements**: `[1,1,1]`
4. **Duplicates**: `[1,2,2,3]`
5. **Negative numbers**: `[-1, 0, 1]`
6. **Integer overflow**: `sum` of large numbers
7. **Null/None**: Missing nodes, null pointers

## Time Management (45-min interview)

| Phase | Time | Activities |
|-------|------|------------|
| Understand | 5 min | Questions, examples |
| Plan | 5 min | Pattern matching, approach |
| Implement | 20 min | Clean code |
| Test | 10 min | Trace, edge cases |
| Buffer | 5 min | Questions, discussion |
