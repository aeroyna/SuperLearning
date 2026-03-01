## The Greedy Framework

A greedy algorithm is an approach for solving optimization problems by making the choice that seems best at the moment. At each step, it makes a locally optimal choice with the hope that this series of local optima will lead to a globally optimal solution.

### The Core Idea
The greedy strategy is built on a simple, intuitive concept: to get the best overall result, consistently make the best possible choice *right now*. This approach doesn't work for all problems, but when it does, it often leads to simpler and more efficient solutions than other techniques like Dynamic Programming.

For a problem to be solvable with a greedy approach, it must exhibit two key properties:

1.  **Greedy Choice Property**: A globally optimal solution can be arrived at by selecting a locally optimal choice. The choice made at each step (the greedy choice) must be part of some globally optimal solution. You have to prove that the greedy choice is always "safe" and will not prevent you from reaching the best final outcome.
2.  **Optimal Substructure**: An optimal solution to the overall problem contains within it optimal solutions to its subproblems. This is a property that greedy algorithms share with Dynamic Programming.

### Proving Correctness
The most difficult part of designing a greedy algorithm is **proving that it is correct**. Simply feeling that the strategy works is not enough. A typical proof involves a "proof by contradiction" or an "exchange argument":
1.  Assume there is an optimal solution that is better than the solution found by the greedy algorithm.
2.  Show that if you take the first step where the greedy choice differs from the optimal solution's choice, you can swap the greedy choice in without making the optimal solution worse.
3.  By repeating this process, you can transform the optimal solution into the greedy solution without changing its quality, thus proving that the greedy solution is also optimal.

### General Framework
1.  **Identify the Greedy Choice**: Figure out what a single, locally optimal decision looks like. What makes one choice "greedier" than another? (e.g., picking the activity that finishes earliest, picking the item with the highest value-to-weight ratio).
2.  **Sort (if necessary)**: Many greedy algorithms require sorting the input based on the greedy choice criteria (e.g., sort jobs by finish time, sort items by value).
3.  **Iterate and Build**: Loop through the sorted items, making the greedy choice at each step and building up a solution.

Mastering greedy algorithms is less about memorizing code and more about identifying when a greedy approach is viable and being able to justify why it works.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)