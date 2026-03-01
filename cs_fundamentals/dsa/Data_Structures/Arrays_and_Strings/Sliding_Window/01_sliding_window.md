# Sliding Window Technique

Sliding window is a powerful technique for solving subarray problems efficiently. It's implemented using two pointers that define a "window" over the array.

## Core Concept

Instead of checking all possible subarrays O(n²), we maintain a window that:
- **Expands** by moving the right pointer
- **Shrinks** by moving the left pointer
- **Slides** along the array from left to right

This gives us O(n) time complexity.

## When to Use Sliding Window

Look for problems that:

1. **Define a valid subarray** based on some constraint:
   - Sum less than/equal to k
   - At most k distinct elements
   - Contains specific characters

2. **Ask for optimal subarray**:
   - Longest valid subarray
   - Shortest valid subarray
   - Number of valid subarrays

## Pattern 1: Variable Size Window

The window size changes based on constraints.

### Template

> [!example]- C++
> ```cpp
> int slidingWindowVariable(vector<int>& arr, int constraint) {
>     int left = 0;
>     int result = 0;
>     int windowState = 0;  // Track sum, count, etc.
>
>     for (int right = 0; right < arr.size(); right++) {
>         // Expand: Add arr[right] to window
>         windowState += arr[right];
>
>         // Shrink: While window is invalid
>         while (windowIsInvalid(windowState, constraint)) {
>             windowState -= arr[left];
>             left++;
>         }
>
>         // Update result (window is now valid)
>         result = max(result, right - left + 1);
>     }
>
>     return result;
> }
> ```

> [!example]- Java
> ```java
> public int slidingWindowVariable(int[] arr, int constraint) {
>     int left = 0;
>     int result = 0;
>     int windowState = 0;  // Track sum, count, etc.
>
>     for (int right = 0; right < arr.length; right++) {
>         // Expand: Add arr[right] to window
>         windowState += arr[right];
>
>         // Shrink: While window is invalid
>         while (windowIsInvalid(windowState, constraint)) {
>             windowState -= arr[left];
>             left++;
>         }
>
>         // Update result (window is now valid)
>         result = Math.max(result, right - left + 1);
>     }
>
>     return result;
> }
> ```

> [!example]- Python
> ```python
> def sliding_window_variable(arr, constraint):
>     left = 0
>     result = 0
>     window_state = 0  # Track sum, count, etc.
>
>     for right in range(len(arr)):
>         # Expand: Add arr[right] to window
>         window_state += arr[right]
>
>         # Shrink: While window is invalid
>         while window_is_invalid(window_state, constraint):
>             window_state -= arr[left]
>             left += 1
>
>         # Update result (window is now valid)
>         result = max(result, right - left + 1)
>
>     return result
> ```

> [!example]- JavaScript
> ```javascript
> function slidingWindowVariable(arr, constraint) {
>     let left = 0;
>     let result = 0;
>     let windowState = 0;  // Track sum, count, etc.
>
>     for (let right = 0; right < arr.length; right++) {
>         // Expand: Add arr[right] to window
>         windowState += arr[right];
>
>         // Shrink: While window is invalid
>         while (windowIsInvalid(windowState, constraint)) {
>             windowState -= arr[left];
>             left++;
>         }
>
>         // Update result (window is now valid)
>         result = Math.max(result, right - left + 1);
>     }
>
>     return result;
> }
> ```

### Example: Longest Subarray with Sum <= k

> [!example]- C++
> ```cpp
> int longestSubarray(vector<int>& nums, int k) {
>     int left = 0;
>     int currentSum = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < nums.size(); right++) {
>         currentSum += nums[right];
>
>         while (currentSum > k) {
>             currentSum -= nums[left];
>             left++;
>         }
>
>         maxLength = max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Java
> ```java
> public int longestSubarray(int[] nums, int k) {
>     int left = 0;
>     int currentSum = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < nums.length; right++) {
>         currentSum += nums[right];
>
>         while (currentSum > k) {
>             currentSum -= nums[left];
>             left++;
>         }
>
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Python
> ```python
> def longestSubarray(nums, k):
>     left = 0
>     current_sum = 0
>     max_length = 0
>
>     for right in range(len(nums)):
>         current_sum += nums[right]
>
>         while current_sum > k:
>             current_sum -= nums[left]
>             left += 1
>
>         max_length = max(max_length, right - left + 1)
>
>     return max_length
> ```

> [!example]- JavaScript
> ```javascript
> function longestSubarray(nums, k) {
>     let left = 0;
>     let currentSum = 0;
>     let maxLength = 0;
>
>     for (let right = 0; right < nums.length; right++) {
>         currentSum += nums[right];
>
>         while (currentSum > k) {
>             currentSum -= nums[left];
>             left++;
>         }
>
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

### Example: Longest Substring Without Repeating Characters

> [!example]- C++
> ```cpp
> int lengthOfLongestSubstring(string s) {
>     unordered_set<char> charSet;
>     int left = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < s.length(); right++) {
>         while (charSet.count(s[right])) {
>             charSet.erase(s[left]);
>             left++;
>         }
>
>         charSet.insert(s[right]);
>         maxLength = max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Java
> ```java
> public int lengthOfLongestSubstring(String s) {
>     Set<Character> charSet = new HashSet<>();
>     int left = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < s.length(); right++) {
>         while (charSet.contains(s.charAt(right))) {
>             charSet.remove(s.charAt(left));
>             left++;
>         }
>
>         charSet.add(s.charAt(right));
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Python
> ```python
> def lengthOfLongestSubstring(s):
>     char_set = set()
>     left = 0
>     max_length = 0
>
>     for right in range(len(s)):
>         while s[right] in char_set:
>             char_set.remove(s[left])
>             left += 1
>
>         char_set.add(s[right])
>         max_length = max(max_length, right - left + 1)
>
>     return max_length
> ```

> [!example]- JavaScript
> ```javascript
> function lengthOfLongestSubstring(s) {
>     const charSet = new Set();
>     let left = 0;
>     let maxLength = 0;
>
>     for (let right = 0; right < s.length; right++) {
>         while (charSet.has(s[right])) {
>             charSet.delete(s[left]);
>             left++;
>         }
>
>         charSet.add(s[right]);
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

### Example: Minimum Window Substring

> [!example]- C++
> ```cpp
> string minWindow(string s, string t) {
>     unordered_map<char, int> need, have;
>     for (char c : t) need[c]++;
>
>     int haveCount = 0;
>     int needCount = need.size();
>     string result = "";
>     int resultLen = INT_MAX;
>     int left = 0;
>
>     for (int right = 0; right < s.length(); right++) {
>         char c = s[right];
>         have[c]++;
>
>         if (need.count(c) && have[c] == need[c]) {
>             haveCount++;
>         }
>
>         while (haveCount == needCount) {
>             // Update result
>             if (right - left + 1 < resultLen) {
>                 resultLen = right - left + 1;
>                 result = s.substr(left, resultLen);
>             }
>
>             // Shrink window
>             char leftChar = s[left];
>             have[leftChar]--;
>             if (need.count(leftChar) && have[leftChar] < need[leftChar]) {
>                 haveCount--;
>             }
>             left++;
>         }
>     }
>
>     return result;
> }
> ```

> [!example]- Java
> ```java
> public String minWindow(String s, String t) {
>     Map<Character, Integer> need = new HashMap<>();
>     Map<Character, Integer> have = new HashMap<>();
>
>     for (char c : t.toCharArray()) {
>         need.put(c, need.getOrDefault(c, 0) + 1);
>     }
>
>     int haveCount = 0;
>     int needCount = need.size();
>     String result = "";
>     int resultLen = Integer.MAX_VALUE;
>     int left = 0;
>
>     for (int right = 0; right < s.length(); right++) {
>         char c = s.charAt(right);
>         have.put(c, have.getOrDefault(c, 0) + 1);
>
>         if (need.containsKey(c) && have.get(c).equals(need.get(c))) {
>             haveCount++;
>         }
>
>         while (haveCount == needCount) {
>             // Update result
>             if (right - left + 1 < resultLen) {
>                 resultLen = right - left + 1;
>                 result = s.substring(left, right + 1);
>             }
>
>             // Shrink window
>             char leftChar = s.charAt(left);
>             have.put(leftChar, have.get(leftChar) - 1);
>             if (need.containsKey(leftChar) && have.get(leftChar) < need.get(leftChar)) {
>                 haveCount--;
>             }
>             left++;
>         }
>     }
>
>     return result;
> }
> ```

> [!example]- Python
> ```python
> def minWindow(s, t):
>     from collections import Counter
>
>     need = Counter(t)
>     have = {}
>     have_count = 0
>     need_count = len(need)
>
>     result = ""
>     result_len = float('inf')
>     left = 0
>
>     for right in range(len(s)):
>         char = s[right]
>         have[char] = have.get(char, 0) + 1
>
>         if char in need and have[char] == need[char]:
>             have_count += 1
>
>         while have_count == need_count:
>             # Update result
>             if right - left + 1 < result_len:
>                 result_len = right - left + 1
>                 result = s[left:right + 1]
>
>             # Shrink window
>             left_char = s[left]
>             have[left_char] -= 1
>             if left_char in need and have[left_char] < need[left_char]:
>                 have_count -= 1
>             left += 1
>
>     return result
> ```

> [!example]- JavaScript
> ```javascript
> function minWindow(s, t) {
>     const need = new Map();
>     const have = new Map();
>
>     for (const c of t) {
>         need.set(c, (need.get(c) || 0) + 1);
>     }
>
>     let haveCount = 0;
>     const needCount = need.size;
>     let result = "";
>     let resultLen = Infinity;
>     let left = 0;
>
>     for (let right = 0; right < s.length; right++) {
>         const c = s[right];
>         have.set(c, (have.get(c) || 0) + 1);
>
>         if (need.has(c) && have.get(c) === need.get(c)) {
>             haveCount++;
>         }
>
>         while (haveCount === needCount) {
>             // Update result
>             if (right - left + 1 < resultLen) {
>                 resultLen = right - left + 1;
>                 result = s.substring(left, right + 1);
>             }
>
>             // Shrink window
>             const leftChar = s[left];
>             have.set(leftChar, have.get(leftChar) - 1);
>             if (need.has(leftChar) && have.get(leftChar) < need.get(leftChar)) {
>                 haveCount--;
>             }
>             left++;
>         }
>     }
>
>     return result;
> }
> ```

## Pattern 2: Fixed Size Window

The window has a constant size k.

### Template

> [!example]- C++
> ```cpp
> int slidingWindowFixed(vector<int>& arr, int k) {
>     // Build first window
>     int windowState = 0;
>     for (int i = 0; i < k; i++) {
>         windowState += arr[i];
>     }
>
>     int result = windowState;
>
>     // Slide window
>     for (int i = k; i < arr.size(); i++) {
>         windowState += arr[i];      // Add new element
>         windowState -= arr[i - k];  // Remove old element
>         result = max(result, windowState);
>     }
>
>     return result;
> }
> ```

> [!example]- Java
> ```java
> public int slidingWindowFixed(int[] arr, int k) {
>     // Build first window
>     int windowState = 0;
>     for (int i = 0; i < k; i++) {
>         windowState += arr[i];
>     }
>
>     int result = windowState;
>
>     // Slide window
>     for (int i = k; i < arr.length; i++) {
>         windowState += arr[i];      // Add new element
>         windowState -= arr[i - k];  // Remove old element
>         result = Math.max(result, windowState);
>     }
>
>     return result;
> }
> ```

> [!example]- Python
> ```python
> def sliding_window_fixed(arr, k):
>     # Build first window
>     window_state = 0
>     for i in range(k):
>         window_state += arr[i]
>
>     result = window_state
>
>     # Slide window
>     for i in range(k, len(arr)):
>         window_state += arr[i]      # Add new element
>         window_state -= arr[i - k]  # Remove old element
>         result = max(result, window_state)
>
>     return result
> ```

> [!example]- JavaScript
> ```javascript
> function slidingWindowFixed(arr, k) {
>     // Build first window
>     let windowState = 0;
>     for (let i = 0; i < k; i++) {
>         windowState += arr[i];
>     }
>
>     let result = windowState;
>
>     // Slide window
>     for (let i = k; i < arr.length; i++) {
>         windowState += arr[i];      // Add new element
>         windowState -= arr[i - k];  // Remove old element
>         result = Math.max(result, windowState);
>     }
>
>     return result;
> }
> ```

### Example: Maximum Average Subarray

> [!example]- C++
> ```cpp
> double findMaxAverage(vector<int>& nums, int k) {
>     int windowSum = 0;
>     for (int i = 0; i < k; i++) {
>         windowSum += nums[i];
>     }
>
>     int maxSum = windowSum;
>
>     for (int i = k; i < nums.size(); i++) {
>         windowSum += nums[i] - nums[i - k];
>         maxSum = max(maxSum, windowSum);
>     }
>
>     return (double)maxSum / k;
> }
> ```

> [!example]- Java
> ```java
> public double findMaxAverage(int[] nums, int k) {
>     int windowSum = 0;
>     for (int i = 0; i < k; i++) {
>         windowSum += nums[i];
>     }
>
>     int maxSum = windowSum;
>
>     for (int i = k; i < nums.length; i++) {
>         windowSum += nums[i] - nums[i - k];
>         maxSum = Math.max(maxSum, windowSum);
>     }
>
>     return (double)maxSum / k;
> }
> ```

> [!example]- Python
> ```python
> def findMaxAverage(nums, k):
>     window_sum = sum(nums[:k])
>     max_sum = window_sum
>
>     for i in range(k, len(nums)):
>         window_sum += nums[i] - nums[i - k]
>         max_sum = max(max_sum, window_sum)
>
>     return max_sum / k
> ```

> [!example]- JavaScript
> ```javascript
> function findMaxAverage(nums, k) {
>     let windowSum = 0;
>     for (let i = 0; i < k; i++) {
>         windowSum += nums[i];
>     }
>
>     let maxSum = windowSum;
>
>     for (let i = k; i < nums.length; i++) {
>         windowSum += nums[i] - nums[i - k];
>         maxSum = Math.max(maxSum, windowSum);
>     }
>
>     return maxSum / k;
> }
> ```

### Example: Max Consecutive Ones III

> [!example]- C++
> ```cpp
> int longestOnes(vector<int>& nums, int k) {
>     // Window with at most k zeros
>     int left = 0;
>     int zeroCount = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < nums.size(); right++) {
>         if (nums[right] == 0) {
>             zeroCount++;
>         }
>
>         while (zeroCount > k) {
>             if (nums[left] == 0) {
>                 zeroCount--;
>             }
>             left++;
>         }
>
>         maxLength = max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Java
> ```java
> public int longestOnes(int[] nums, int k) {
>     // Window with at most k zeros
>     int left = 0;
>     int zeroCount = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < nums.length; right++) {
>         if (nums[right] == 0) {
>             zeroCount++;
>         }
>
>         while (zeroCount > k) {
>             if (nums[left] == 0) {
>                 zeroCount--;
>             }
>             left++;
>         }
>
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Python
> ```python
> def longestOnes(nums, k):
>     # Window with at most k zeros
>     left = 0
>     zero_count = 0
>     max_length = 0
>
>     for right in range(len(nums)):
>         if nums[right] == 0:
>             zero_count += 1
>
>         while zero_count > k:
>             if nums[left] == 0:
>                 zero_count -= 1
>             left += 1
>
>         max_length = max(max_length, right - left + 1)
>
>     return max_length
> ```

> [!example]- JavaScript
> ```javascript
> function longestOnes(nums, k) {
>     // Window with at most k zeros
>     let left = 0;
>     let zeroCount = 0;
>     let maxLength = 0;
>
>     for (let right = 0; right < nums.length; right++) {
>         if (nums[right] === 0) {
>             zeroCount++;
>         }
>
>         while (zeroCount > k) {
>             if (nums[left] === 0) {
>                 zeroCount--;
>             }
>             left++;
>         }
>
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

## Pattern 3: Count Subarrays

Count the number of valid subarrays.

### Key Insight

For a valid window `[left, right]`, there are `right - left + 1` valid subarrays **ending at right**.

> [!example]- C++
> ```cpp
> int countValidSubarrays(vector<int>& nums, int constraint) {
>     int left = 0;
>     int count = 0;
>     int windowState = 1;  // or 0 for sum
>
>     for (int right = 0; right < nums.size(); right++) {
>         windowState *= nums[right];  // or += for sum
>
>         while (windowIsInvalid(windowState, constraint)) {
>             windowState /= nums[left];  // or -= for sum
>             left++;
>         }
>
>         // All subarrays ending at right are valid
>         count += right - left + 1;
>     }
>
>     return count;
> }
> ```

> [!example]- Java
> ```java
> public int countValidSubarrays(int[] nums, int constraint) {
>     int left = 0;
>     int count = 0;
>     int windowState = 1;  // or 0 for sum
>
>     for (int right = 0; right < nums.length; right++) {
>         windowState *= nums[right];  // or += for sum
>
>         while (windowIsInvalid(windowState, constraint)) {
>             windowState /= nums[left];  // or -= for sum
>             left++;
>         }
>
>         // All subarrays ending at right are valid
>         count += right - left + 1;
>     }
>
>     return count;
> }
> ```

> [!example]- Python
> ```python
> def countValidSubarrays(nums, constraint):
>     left = 0
>     count = 0
>     window_state = 1  # or 0 for sum
>
>     for right in range(len(nums)):
>         window_state *= nums[right]  # or += for sum
>
>         while window_is_invalid(window_state, constraint):
>             window_state //= nums[left]  # or -= for sum
>             left += 1
>
>         # All subarrays ending at right are valid
>         count += right - left + 1
>
>     return count
> ```

> [!example]- JavaScript
> ```javascript
> function countValidSubarrays(nums, constraint) {
>     let left = 0;
>     let count = 0;
>     let windowState = 1;  // or 0 for sum
>
>     for (let right = 0; right < nums.length; right++) {
>         windowState *= nums[right];  // or += for sum
>
>         while (windowIsInvalid(windowState, constraint)) {
>             windowState = Math.floor(windowState / nums[left]);  // or -= for sum
>             left++;
>         }
>
>         // All subarrays ending at right are valid
>         count += right - left + 1;
>     }
>
>     return count;
> }
> ```

### Example: Subarray Product Less Than K

> [!example]- C++
> ```cpp
> int numSubarrayProductLessThanK(vector<int>& nums, int k) {
>     if (k <= 1) return 0;
>
>     int left = 0;
>     int product = 1;
>     int count = 0;
>
>     for (int right = 0; right < nums.size(); right++) {
>         product *= nums[right];
>
>         while (product >= k) {
>             product /= nums[left];
>             left++;
>         }
>
>         count += right - left + 1;
>     }
>
>     return count;
> }
> ```

> [!example]- Java
> ```java
> public int numSubarrayProductLessThanK(int[] nums, int k) {
>     if (k <= 1) return 0;
>
>     int left = 0;
>     int product = 1;
>     int count = 0;
>
>     for (int right = 0; right < nums.length; right++) {
>         product *= nums[right];
>
>         while (product >= k) {
>             product /= nums[left];
>             left++;
>         }
>
>         count += right - left + 1;
>     }
>
>     return count;
> }
> ```

> [!example]- Python
> ```python
> def numSubarrayProductLessThanK(nums, k):
>     if k <= 1:
>         return 0
>
>     left = 0
>     product = 1
>     count = 0
>
>     for right in range(len(nums)):
>         product *= nums[right]
>
>         while product >= k:
>             product //= nums[left]
>             left += 1
>
>         count += right - left + 1
>
>     return count
> ```

> [!example]- JavaScript
> ```javascript
> function numSubarrayProductLessThanK(nums, k) {
>     if (k <= 1) return 0;
>
>     let left = 0;
>     let product = 1;
>     let count = 0;
>
>     for (let right = 0; right < nums.length; right++) {
>         product *= nums[right];
>
>         while (product >= k) {
>             product = Math.floor(product / nums[left]);
>             left++;
>         }
>
>         count += right - left + 1;
>     }
>
>     return count;
> }
> ```

## Why O(n)?

Even though there's a while loop inside the for loop:
- The right pointer moves n times total
- The left pointer can only move n times total (it never goes backward)
- Total operations: O(2n) = O(n)

This is **amortized analysis**.

## Sliding Window + Hash Map

Many problems combine sliding window with a hash map to track character/element frequencies.

### Example: Longest Substring with At Most K Distinct Characters

> [!example]- C++
> ```cpp
> int lengthOfLongestSubstringKDistinct(string s, int k) {
>     unordered_map<char, int> charCount;
>     int left = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < s.length(); right++) {
>         charCount[s[right]]++;
>
>         while (charCount.size() > k) {
>             charCount[s[left]]--;
>             if (charCount[s[left]] == 0) {
>                 charCount.erase(s[left]);
>             }
>             left++;
>         }
>
>         maxLength = max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Java
> ```java
> public int lengthOfLongestSubstringKDistinct(String s, int k) {
>     Map<Character, Integer> charCount = new HashMap<>();
>     int left = 0;
>     int maxLength = 0;
>
>     for (int right = 0; right < s.length(); right++) {
>         charCount.put(s.charAt(right),
>                       charCount.getOrDefault(s.charAt(right), 0) + 1);
>
>         while (charCount.size() > k) {
>             charCount.put(s.charAt(left), charCount.get(s.charAt(left)) - 1);
>             if (charCount.get(s.charAt(left)) == 0) {
>                 charCount.remove(s.charAt(left));
>             }
>             left++;
>         }
>
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

> [!example]- Python
> ```python
> def lengthOfLongestSubstringKDistinct(s, k):
>     char_count = {}
>     left = 0
>     max_length = 0
>
>     for right in range(len(s)):
>         char_count[s[right]] = char_count.get(s[right], 0) + 1
>
>         while len(char_count) > k:
>             char_count[s[left]] -= 1
>             if char_count[s[left]] == 0:
>                 del char_count[s[left]]
>             left += 1
>
>         max_length = max(max_length, right - left + 1)
>
>     return max_length
> ```

> [!example]- JavaScript
> ```javascript
> function lengthOfLongestSubstringKDistinct(s, k) {
>     const charCount = new Map();
>     let left = 0;
>     let maxLength = 0;
>
>     for (let right = 0; right < s.length; right++) {
>         charCount.set(s[right], (charCount.get(s[right]) || 0) + 1);
>
>         while (charCount.size > k) {
>             charCount.set(s[left], charCount.get(s[left]) - 1);
>             if (charCount.get(s[left]) === 0) {
>                 charCount.delete(s[left]);
>             }
>             left++;
>         }
>
>         maxLength = Math.max(maxLength, right - left + 1);
>     }
>
>     return maxLength;
> }
> ```

## Practice Problems

| Problem | Type | Difficulty |
|---------|------|------------|
| Maximum Average Subarray I | Fixed | Easy |
| Max Consecutive Ones III | Variable | Medium |
| Longest Substring Without Repeating | Variable | Medium |
| Minimum Window Substring | Variable | Hard |
| Sliding Window Maximum | Fixed + Deque | Hard |
| Subarray Product Less Than K | Count | Medium |
| Permutation in String | Fixed | Medium |
| Fruit Into Baskets | Variable | Medium |
