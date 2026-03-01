# Dynamic Programming

Dynamic Programming (DP) is a powerful technique for solving optimization and counting problems by breaking them down into simpler subproblems.

The key idea of DP is to solve each subproblem only once and store the result in a table (or memoization array) to avoid re-computation.

A problem can be solved using DP if it has two properties:

1.  **Optimal Substructure:** An optimal solution to the problem can be constructed from optimal solutions to its subproblems.
2.  **Overlapping Subproblems:** The problem can be broken down into subproblems that are reused several times.

## Example: Fibonacci Sequence

The Fibonacci sequence is a classic example of a problem with overlapping subproblems. A naive recursive solution is very inefficient:

```cpp
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2); // fib(n-2) is computed multiple times
}
```
`fib(5)` calls `fib(4)` and `fib(3)`. `fib(4)` calls `fib(3)` and `fib(2)`. The subproblem `fib(3)` is computed twice.

## Two Approaches to Dynamic Programming

### 1. Memoization (Top-Down)

Memoization is a technique where you store the results of expensive function calls and return the cached result when the same inputs occur again. It's a top-down approach that uses recursion.

#### Fibonacci with Memoization

```cpp
#include <vector>

std::vector<int> memo;

int fib_memo(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n]; // return cached result
    
    memo[n] = fib_memo(n - 1) + fib_memo(n - 2); // cache the result
    return memo[n];
}

int main() {
    int n = 10;
    memo.assign(n + 1, -1);
    int result = fib_memo(n);
    // ...
}
```

### 2. Tabulation (Bottom-Up)

Tabulation is a technique where you solve the problem in a bottom-up fashion. You start by solving the smallest subproblems and build up to the final solution. This approach is iterative and avoids recursion.

#### Fibonacci with Tabulation

```cpp
int fib_tab(int n) {
    if (n <= 1) return n;
    std::vector<int> table(n + 1);
    table[0] = 0;
    table[1] = 1;

    for (int i = 2; i <= n; ++i) {
        table[i] = table[i - 1] + table[i - 2];
    }
    return table[n];
}
```
You can often optimize the space complexity of a tabulation solution if you only need the results of the last few subproblems. For Fibonacci, you only need the last two values:

```cpp
int fib_tab_optimized(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; ++i) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
```

## Common DP Problems

*   **0/1 Knapsack Problem:** Given a set of items, each with a weight and a value, determine the number of each item to include in a collection so that the total weight is less than or equal to a given limit and the total value is as large as possible.
*   **Longest Common Subsequence (LCS):** Find the longest subsequence common to two sequences.
*   **Longest Increasing Subsequence (LIS):** Find the length of the longest subsequence of a given sequence such that all elements of the subsequence are sorted in increasing order.
*   **Edit Distance:** Find the minimum number of operations (insertion, deletion, substitution) required to change one string into another.
*   **Coin Change Problem:** Find the number of ways to make change for a particular amount of money using a given set of coin denominations.

Dynamic programming is a fundamental topic in algorithm design and is frequently asked about in technical interviews.
