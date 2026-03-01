## The Stages of a Coding Interview

A technical interview is more than just solving a problem; it's a structured conversation designed to evaluate your technical skills, problem-solving process, and communication abilities. Understanding the typical stages can help you navigate it effectively.

### Stage 1: Introductions (~2-3 minutes)
- **Your Goal**: Make a positive first impression.
- **Action**: Have a concise, 30-60 second introduction ready. Cover your background, key experiences, and technical interests.
- **Pro Tip**: Listen actively when the interviewer introduces themselves. Note their role and team—this is valuable for asking insightful questions later.

### Stage 2: Problem Statement & Clarification (~5 minutes)
- **Your Goal**: Ensure you 100% understand the problem before you start coding.
- **Actions**:
    1. **Paraphrase**: Restate the problem in your own words to confirm your understanding.
    2. **Clarify Constraints**: Ask about the size of the input (e.g., "How large can `n` be?"). This hints at the required time complexity.
    3. **Discuss Edge Cases**: Ask about empty inputs, negative numbers, duplicates, or other edge cases. ("What should happen if the input array is empty?").
    4. **Walk Through an Example**: Use a small example provided by the interviewer (or create your own) and walk through the expected input and output.

### Stage 3: Brainstorming & High-Level Approach (~5-10 minutes)
- **Your Goal**: Develop and communicate a viable plan before writing code.
- **Actions**:
    1. **Think Aloud**: This is critical. Verbalize your thought process. Start with a brute-force solution ("A straightforward way would be to use nested loops...") and then discuss how to optimize it.
    2. **Identify Patterns**: Connect the problem to known data structures and algorithms ("This seems like it could be a graph problem," or "Since we need the top K items, a heap could be useful.").
    3. **Propose a Solution**: Before coding, outline your final approach to the interviewer. ("My plan is to use a hash map to count frequencies, then a min-heap to find the top K elements. This should give us O(n log k) time complexity."). Get their buy-in.

### Stage 4: Implementation (Coding) (~15-20 minutes)
- **Your Goal**: Write clean, correct, and readable code.
- **Actions**:
    1. **Communicate As You Code**: Briefly explain what you're doing ("Now I'm initializing the hash map to store counts," "This helper function will handle the DFS traversal.").
    2. **Write Clean Code**: Use meaningful variable names. Break down complex logic into helper functions if necessary.
    3. **Start Simple**: Don't over-optimize prematurely. Write a clear, working solution first.

### Stage 5: Testing & Debugging (~5 minutes)
- **Your Goal**: Verify your code's correctness and demonstrate a good testing mindset.
- **Actions**:
    1. **Test with Examples**: Don't just say "I think it works." Manually trace your code with a simple example and the edge cases you discussed earlier.
    2. **Track Variables**: Keep track of the state of key variables as you trace the code.
    3. **Find and Fix Bugs**: If you find a bug, explain what's wrong and how you plan to fix it. This is a chance to showcase your debugging skills.

### Stage 6: Complexity Analysis & Follow-ups (~3-5 minutes)
- **Your Goal**: Analyze your solution's performance and discuss alternatives.
- **Actions**:
    - State the **Time Complexity** and **Space Complexity** of your solution. Explain *why* (e.g., "It's O(n log n) because the sorting step dominates the runtime.").
    - Be prepared for follow-up questions like, "Can you do better?" or "What if the input data was too large to fit in memory?".

### Stage 7: Your Questions & Outro (~3-5 minutes)
- **Your Goal**: Show your interest in the role and the company.
- **Actions**:
    - Ask thoughtful questions. Avoid questions easily answered by a Google search. Ask about the team's challenges, the tech stack, company culture, or the interviewer's own experience.
    - End on a positive and professional note.
