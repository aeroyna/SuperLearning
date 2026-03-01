## A Framework for Solving Dynamic Programming Problems

Dynamic Programming (DP) is a powerful technique for solving optimization and counting problems by breaking them down into simpler, overlapping subproblems. While DP problems can be daunting, most can be solved by following a systematic framework.

A problem might be a good candidate for DP if it exhibits two key properties:
1.  **Overlapping Subproblems**: The problem can be broken down into subproblems that are reused multiple times.
2.  **Optimal Substructure**: The optimal solution to the overall problem can be constructed from the optimal solutions of its subproblems.

### The DP Framework

1.  **Define the State**: This is the most crucial step. Identify the parameters that uniquely define a subproblem. The "state" is what you are trying to compute. For example, `dp[i]` could represent "the maximum profit up to day `i`" or "the number of ways to reach step `i`". For 2D DP, the state might be `dp[i][j]`, representing a subproblem on a grid or with two inputs.

2.  **Find the Recurrence Relation**: This is the core logic. You must define the state for a given step in terms of the states of previous steps. Ask yourself: "How can I solve for `dp[i]` if I already know the answers for `dp[i-1]`, `dp[i-2]`, etc.?" This relation formalizes the optimal substructure property.

3.  **Identify the Base Cases**: Determine the smallest subproblems that can be solved directly, without recursion. These are the starting points for your recurrence relation. For example, `dp[0]` might be the value of the first element, or 0.

### Two Main Approaches: Top-Down vs. Bottom-Up

Once you have defined the state, recurrence, and base cases, you can implement the solution in one of two ways:

#### 1. Top-Down with Memoization
This is a recursive approach. You write a function that computes the state and relies on recursive calls to solve the subproblems. To avoid re-computing the same subproblem, you store (or "memoize") the result of each state in a cache (e.g., a hash map or an array). Before computing a state, you check if it's already in the cache.

- **Pros**: Often more intuitive and closer to the pure recursive definition. It only computes the subproblems that are actually needed.
- **Cons**: Can lead to recursion depth errors for very large inputs.

#### 2. Bottom-Up with Tabulation
This is an iterative approach. You create a DP table (usually an array or matrix) and fill it up, starting from the base cases and iterating towards the final solution. Each cell `dp[i]` is filled using the values of previously computed cells according to the recurrence relation.

- **Pros**: No recursion overhead, generally faster. Avoids stack overflow issues.
- **Cons**: Can be less intuitive to formulate. It may compute all possible subproblems, even if some are not strictly necessary.

For most interviews, either approach is acceptable, but being able to explain and implement both is a huge plus. The bottom-up approach is often considered slightly more optimized.
