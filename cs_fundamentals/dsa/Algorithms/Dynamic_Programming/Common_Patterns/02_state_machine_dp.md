## State Machine DP

A number of advanced Dynamic Programming problems can be modeled using a **Finite State Machine (FSM)**. In this pattern, the DP state doesn't just track an index or two, but also the "state" of the world at that point in time (e.g., "am I currently holding a stock?"). The recurrence relations then become transitions between these states.

This pattern is particularly useful for problems where the decision at step `i` is constrained by the action taken at step `i-1`.

### The Core Idea
1.  **Identify the States**: Determine the different possible situations or "states" you can be in at any given step. Common states include:
    - `Holding` vs. `Not Holding` a stock.
    - `Buying`, `Selling`, or `Cooling Down`.
    - Being in a certain state in a sequence.

2.  **Define the DP State**: Your DP state will typically be `dp[i][state]`, representing the maximum profit/best solution at step `i`, given that you are in a particular `state`.

3.  **Map the Transitions**: Draw out the state machine diagram. For each state, determine what actions can lead you to other states. These transitions become your recurrence relations.
    - To be in state `A` today, from which states could I have come from yesterday?
    - If I was in state `B` yesterday, what action can I take today to land in state `A`?

### Classic Example: Best Time to Buy and Sell Stock with Cooldown (LeetCode #309)
**Problem**: You are given an array `prices`. You may complete as many transactions as you like, but with one constraint: after you sell a stock, you cannot buy on the next day (i.e., there is a 1-day cooldown). Find the maximum profit.

**State Machine Approach**:
At any given day `i`, we can be in one of three states:
1.  **`held`**: We own a stock.
2.  **`sold`**: We have just sold a stock today.
3.  **`reset` / `cooldown`**: We do not own a stock and are free to buy one.

**State Definitions**:
- `dp[i][held]`: The maximum profit at the end of day `i`, given that we are holding a stock.
- `dp[i][sold]`: The maximum profit at the end of day `i`, given that we sold a stock on this day.
- `dp[i][reset]`: The maximum profit at the end of day `i`, given that we are in a "reset" state (we don't own a stock and are not in cooldown from a sale on day `i`).

**Transitions (Recurrence Relations)**:
- To be **`held`** today, you either:
  1.  Were `held` yesterday and did nothing: `dp[i-1][held]`.
  2.  Were in a `reset` state yesterday and bought today: `dp[i-1][reset] - prices[i]`.
  `dp[i][held] = max(dp[i-1][held], dp[i-1][reset] - prices[i])`

- To have **`sold`** today, you must have been `held` yesterday:
  `dp[i][sold] = dp[i-1][held] + prices[i]`

- To be in a **`reset`** state today, you either:
  1.  Were in `reset` yesterday and did nothing.
  2.  Just sold yesterday and are now "resting".
  `dp[i][reset] = max(dp[i-1][reset], dp[i-1][sold])`

**Final Answer**: The maximum profit is the max of the `sold` and `reset` states on the final day, as you can't end in a `held` state and have realized profit.

This state machine approach elegantly handles the "cooldown" constraint by creating an explicit `sold` state that you must pass through, forcing a transition to `reset` on the next day. It transforms a complex set of rules into a clear set of DP transitions.
