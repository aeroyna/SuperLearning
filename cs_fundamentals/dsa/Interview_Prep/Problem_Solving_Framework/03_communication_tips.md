## Communication Tips for Coding Interviews

In a technical interview, how you communicate is just as important as the code you write. An interviewer is not just looking for a correct answer; they are evaluating if you would be a good colleague. They want to see how you think, how you collaborate, and how you handle challenges.

### The #1 Rule: Think Out Loud
This is the most critical communication skill. The interviewer cannot read your mind. If you are silent for long periods, they will assume you are stuck. By verbalizing your thought process, you allow the interviewer to:
- Understand your approach.
- Offer hints or correct misunderstandings if you're going down the wrong path.
- Evaluate your problem-solving ability even if you don't reach a perfect final solution.

**What to say**:
- "My initial thought is a brute-force approach where we check every possibility..."
- "This makes me think of a graph problem, where the cities are nodes and roads are edges."
- "A hash map seems like a good choice here because we need fast lookups."
- "I'm considering a sliding window approach to avoid nested loops."
- "I'm a little stuck on how to handle this edge case. Let me think about..."

### Before You Code: Clarify and Plan
Never jump straight into coding.
- **Ask Clarifying Questions**: Confirm your understanding of the inputs, outputs, and constraints. This shows you are careful and methodical.
- **State Your High-Level Plan**: Before writing a single line, explain your chosen algorithm and its complexity. Get confirmation from the interviewer. "I'm going to sort the array first, then use two pointers. This should be O(n log n). Does that sound like a reasonable approach to you?"

### While You Code: Narrate Your Actions
Keep a running commentary as you implement your solution. It doesn't have to be a detailed explanation of every line, but it should cover the main parts.
- "First, I'll initialize my distance array to infinity."
- "Now, I'm writing a helper function to do the DFS traversal."
- "I'm using a `deque` here because I need an efficient queue for my BFS."

### When You're Stuck: Be Proactive
It's completely normal to get stuck. How you handle it is what matters.
- **Don't go silent**: The worst thing you can do is stop talking.
- **Verbalize the problem**: "I'm having trouble figuring out the exact condition for my `while` loop," or "I know I need to handle this case, but I'm not sure of the most efficient way."
- **Simplify**: "Could I try solving it for a simpler case first? For example, if there were no duplicates."
- **Use the Interviewer**: You can ask for a hint, but do it strategically. "I'm thinking about either a recursive or iterative solution here, do you have a preference or a hint about which might be more fruitful?"

### After You Code: Test and Analyze
- **Don't just say "I'm done."**: Start testing your code immediately.
- **Walk through an example**: Choose a non-trivial example and trace your code's execution, tracking the values of key variables.
- **Analyze Complexity**: Clearly state the time and space complexity and be ready to justify your analysis.
- **Discuss Trade-offs**: If you made a choice (e.g., used more space for a faster runtime), explain why you made it.
