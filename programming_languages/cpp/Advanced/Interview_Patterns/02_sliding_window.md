# Sliding Window

The sliding window technique maintains a "window" over a contiguous subarray/substring to solve problems efficiently.

## Types of Sliding Window

### 1. Fixed Size Window

Window size is predetermined.

```cpp
// Maximum sum of subarray of size k
int maxSumSubarray(std::vector<int>& nums, int k) {
    int n = nums.size();
    if (n < k) return -1;

    // Calculate sum of first window
    int windowSum = 0;
    for (int i = 0; i < k; ++i) {
        windowSum += nums[i];
    }

    int maxSum = windowSum;

    // Slide the window
    for (int i = k; i < n; ++i) {
        windowSum += nums[i] - nums[i - k];  // Add new, remove old
        maxSum = std::max(maxSum, windowSum);
    }

    return maxSum;
}
```

### 2. Variable Size Window

Window expands and shrinks based on conditions.

```cpp
// Minimum window containing sum >= target
int minSubArrayLen(int target, std::vector<int>& nums) {
    int left = 0, sum = 0;
    int minLen = INT_MAX;

    for (int right = 0; right < nums.size(); ++right) {
        sum += nums[right];  // Expand window

        while (sum >= target) {
            minLen = std::min(minLen, right - left + 1);
            sum -= nums[left];  // Shrink window
            ++left;
        }
    }

    return minLen == INT_MAX ? 0 : minLen;
}
```

## Classic Problems

### Longest Substring Without Repeating Characters

```cpp
int lengthOfLongestSubstring(const std::string& s) {
    std::unordered_set<char> window;
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.size(); ++right) {
        while (window.count(s[right])) {
            window.erase(s[left]);
            ++left;
        }
        window.insert(s[right]);
        maxLen = std::max(maxLen, right - left + 1);
    }

    return maxLen;
}

// Alternative with hash map (faster)
int lengthOfLongestSubstring(const std::string& s) {
    std::unordered_map<char, int> lastSeen;
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.size(); ++right) {
        if (lastSeen.count(s[right]) && lastSeen[s[right]] >= left) {
            left = lastSeen[s[right]] + 1;
        }
        lastSeen[s[right]] = right;
        maxLen = std::max(maxLen, right - left + 1);
    }

    return maxLen;
}
```

### Minimum Window Substring

```cpp
std::string minWindow(std::string s, std::string t) {
    if (t.empty() || s.size() < t.size()) return "";

    std::unordered_map<char, int> need, have;
    for (char c : t) ++need[c];

    int left = 0, minLen = INT_MAX, minStart = 0;
    int required = need.size(), formed = 0;

    for (int right = 0; right < s.size(); ++right) {
        char c = s[right];
        ++have[c];

        if (need.count(c) && have[c] == need[c]) {
            ++formed;
        }

        while (formed == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minStart = left;
            }

            char leftChar = s[left];
            --have[leftChar];
            if (need.count(leftChar) && have[leftChar] < need[leftChar]) {
                --formed;
            }
            ++left;
        }
    }

    return minLen == INT_MAX ? "" : s.substr(minStart, minLen);
}
```

### Maximum of All Subarrays of Size K (Using Deque)

```cpp
std::vector<int> maxSlidingWindow(std::vector<int>& nums, int k) {
    std::deque<int> dq;  // Stores indices
    std::vector<int> result;

    for (int i = 0; i < nums.size(); ++i) {
        // Remove indices outside window
        while (!dq.empty() && dq.front() <= i - k) {
            dq.pop_front();
        }

        // Remove smaller elements (they'll never be max)
        while (!dq.empty() && nums[dq.back()] < nums[i]) {
            dq.pop_back();
        }

        dq.push_back(i);

        if (i >= k - 1) {
            result.push_back(nums[dq.front()]);
        }
    }

    return result;
}
```

### Longest Substring with At Most K Distinct Characters

```cpp
int lengthOfLongestSubstringKDistinct(const std::string& s, int k) {
    std::unordered_map<char, int> count;
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.size(); ++right) {
        ++count[s[right]];

        while (count.size() > k) {
            if (--count[s[left]] == 0) {
                count.erase(s[left]);
            }
            ++left;
        }

        maxLen = std::max(maxLen, right - left + 1);
    }

    return maxLen;
}
```

## Sliding Window Template

```cpp
int slidingWindowTemplate(std::vector<int>& nums) {
    int left = 0;
    int result = 0;  // or INT_MAX/INT_MIN depending on problem

    for (int right = 0; right < nums.size(); ++right) {
        // 1. Add nums[right] to window state

        // 2. While window is invalid, shrink from left
        while (/* window condition not met */) {
            // Remove nums[left] from window state
            ++left;
        }

        // 3. Update result
        result = /* update based on current window */;
    }

    return result;
}
```

## When to Use Sliding Window

- **Contiguous** subarray/substring problems
- Finding **min/max** length subarray with property
- **Sum** or **count** of elements in range
- String problems with **character frequency**
- Problems with "at most K" or "exactly K" constraint

## Key Takeaways

- Fixed window: size is constant
- Variable window: expand right, shrink left
- Often uses hash map for frequency counting
- Time complexity: O(n) - each element visited at most twice
- Space complexity: O(k) or O(1) for fixed alphabet

## Common Interview Questions

> [!question]- What's the difference between two pointer and sliding window?
> Sliding window is a specific application of two pointers focused on contiguous subarrays. Two pointer can work with non-contiguous elements (like sorted array pairs).

> [!question]- How do you handle "exactly K" constraints?
> Often solved as: atMost(K) - atMost(K-1) = exactly(K)
