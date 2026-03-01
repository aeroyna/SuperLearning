# Practice Problems: Greedy Algorithms

Local optimal choice leading to global optimum.

## Interval Greedy

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Jump Game](https://leetcode.com/problems/jump-game/) | Medium | Track `max_reachable` index. |
| [Jump Game II](https://leetcode.com/problems/jump-game-ii/) | Medium | Implicit BFS. `end` of current jump range determines steps. |
| [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Sort by end time. Keep earliest end. |

## Other Greedy

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Gas Station](https://leetcode.com/problems/gas-station/) | Medium | If total gas >= cost, solution exists. Track current tank, reset if < 0. |
| [Partition Labels](https://leetcode.com/problems/partition-labels/) | Medium | Last occurrence index of chars. Extend window to max last index. |
