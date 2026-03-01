# Practice Problems: Arrays and Strings

Master array manipulation, sliding window, two pointers, and string techniques.

## Two Pointers

| Problem                                                                               | Difficulty | Key Insight                                                                             |
| ------------------------------------------------------------------------------------- | ---------- | --------------------------------------------------------------------------------------- |
| [Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)         | Medium     | Sorted array allows moving pointers based on sum comparison.                            |
| [3Sum](https://leetcode.com/problems/3sum/)                                           | Medium     | Fix one number, then use 2-sum (two pointers) on the rest. Handle duplicates.           |
| [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium     | Move the pointer with the shorter height to try and find a taller one.                  |
| [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)             | Hard       | Precompute max-left and max-right, or use two pointers with `max_left` and `max_right`. |

## Sliding Window

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | Contract window when a duplicate is found (move left pointer). |
| [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | Expand to satisfy condition, then contract to minimize length. |
| [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | `window_len - max_freq <= k` condition. |

## Prefix Sum

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium | `sum[i, j] = prefix[j] - prefix[i-1]`. Use hash map to store prefix counts. |
| [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) | Medium | Prefix product * Suffix product. |

## String Manipulation

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium | Sort string or count chars to use as hash map key. |
| [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy | Filter non-alphanumeric, then check reverse or two pointers. |
