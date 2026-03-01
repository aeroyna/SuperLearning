## Binary Search on the Solution Space

Binary search is not just for finding elements in sorted arrays. A more advanced and powerful application is to **search for an answer** within a continuous or discrete range of possible solutions. This technique is often called "Binary Search on the Answer."

### The Core Idea
This pattern is applicable to a specific class of optimization problems, typically those that ask for the **minimum possible maximum** or the **maximum possible minimum** value that satisfies a certain condition.

The key is to recognize that if a certain value `x` is a possible answer, then any value "better" than `x` (e.g., any value `> x` for a maximization problem) might also be possible, while any value "worse" than `x` is definitely not. This creates a monotonic property on the *solution space* itself, allowing us to use binary search.

The process is:
1.  **Define the Search Space**: Determine the absolute minimum (`low`) and maximum (`high`) possible values for the answer.
2.  **Define a `check(value)` function**: This is the most critical part. You must create a function that can determine if it's *possible* to achieve a solution with the given `value`. This `check` function is often implemented using a greedy approach and typically runs in O(n) time.
3.  **Binary Search**:
    -   Perform a binary search on the range `[low, high]`.
    -   In each step, use `check(mid)` to see if the middle value of your answer range is a feasible solution.
    -   If `check(mid)` is `True`, it means `mid` is a potential answer. You then try to find an even better one by narrowing the search space (e.g., for a minimization problem, you'd try smaller values by setting `high = mid - 1`).
    -   If `check(mid)` is `False`, `mid` is not a feasible answer, and you must search for a "worse" one (e.g., for a minimization problem, you need to allow a larger value, so `low = mid + 1`).

### Example: Split Array Largest Sum (LeetCode #410)
**Problem**: Given an array `nums` of non-negative integers and an integer `m`, you need to split the array into `m` non-empty continuous subarrays. Write an algorithm to **minimize the largest sum** among these `m` subarrays.

This is a classic "minimize the maximum" problem, perfect for this technique.

**Solution**:
1.  **Search Space**:
    -   The minimum possible answer (`low`) is the largest single element in the array (the case where each element is its own subarray).
    -   The maximum possible answer (`high`) is the sum of all elements in the array (the case where `m=1`).
2.  **`check(max_sum)` function**: We need to check: *Is it possible to split the array into `m` or fewer subarrays such that no subarray's sum exceeds `max_sum`?*
    -   We can check this greedily. Iterate through `nums`, accumulating a `current_sum`. When `current_sum` plus the next element would exceed `max_sum`, we must start a new subarray. We count how many subarrays are required. If `subarrays_needed <= m`, the check passes.

3.  **Binary Search**: We binary search on the `[low, high]` range of possible sums to find the minimum `max_sum` for which `check(max_sum)` returns `True`.

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>#include <numeric>
>using namespace std;
>
>class Solution {
>public:
>    int splitArray(vector<int>& nums, int m) {
>        // Helper function to check if we can split array into m or fewer subarrays
>        // with maximum sum not exceeding max_sum_allowed
>        auto check = [&](long long max_sum_allowed) -> bool {
>            int subarrays_needed = 1;
>            long long current_sum = 0;
>
>            for (int num : nums) {
>                if (current_sum + num > max_sum_allowed) {
>                    subarrays_needed++;
>                    current_sum = num;
>                } else {
>                    current_sum += num;
>                }
>            }
>            return subarrays_needed <= m;
>        };
>
>        // Define search space
>        long long low = *max_element(nums.begin(), nums.end());
>        long long high = accumulate(nums.begin(), nums.end(), 0LL);
>        long long ans = high;
>
>        // Binary search for the minimum valid maximum sum
>        while (low <= high) {
>            long long mid = low + (high - low) / 2;
>
>            if (check(mid)) {
>                // mid is a possible answer. Try to find a smaller one.
>                ans = mid;
>                high = mid - 1;
>            } else {
>                // mid is too small. Need to allow a larger sum.
>                low = mid + 1;
>            }
>        }
>
>        return ans;
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public int splitArray(int[] nums, int m) {
>        // Helper method to check if we can split array into m or fewer subarrays
>        // with maximum sum not exceeding maxSumAllowed
>        boolean check(int[] nums, int m, long maxSumAllowed) {
>            int subarraysNeeded = 1;
>            long currentSum = 0;
>
>            for (int num : nums) {
>                if (currentSum + num > maxSumAllowed) {
>                    subarraysNeeded++;
>                    currentSum = num;
>                } else {
>                    currentSum += num;
>                }
>            }
>            return subarraysNeeded <= m;
>        }
>
>        // Define search space
>        long low = 0;
>        long high = 0;
>        for (int num : nums) {
>            low = Math.max(low, num);
>            high += num;
>        }
>
>        long ans = high;
>
>        // Binary search for the minimum valid maximum sum
>        while (low <= high) {
>            long mid = low + (high - low) / 2;
>
>            if (check(nums, m, mid)) {
>                // mid is a possible answer. Try to find a smaller one.
>                ans = mid;
>                high = mid - 1;
>            } else {
>                // mid is too small. Need to allow a larger sum.
>                low = mid + 1;
>            }
>        }
>
>        return (int) ans;
>    }
>}
>```

>[!example]- Python
>```python
>def split_array(nums, m):
>    def check(max_sum_allowed):
>        subarrays_needed = 1
>        current_sum = 0
>        for num in nums:
>            if current_sum + num > max_sum_allowed:
>                subarrays_needed += 1
>                current_sum = num
>            else:
>                current_sum += num
>        return subarrays_needed <= m
>
>    low, high = max(nums), sum(nums)
>    ans = high
>
>    while low <= high:
>        mid = low + (high - low) // 2
>        if check(mid):
>            # mid is a possible answer. Try to find a smaller one.
>            ans = mid
>            high = mid - 1
>        else:
>            # mid is too small. Need to allow a larger sum.
>            low = mid + 1
>
>    return ans
>```

>[!example]- JavaScript
>```javascript
>function splitArray(nums, m) {
>    // Helper function to check if we can split array into m or fewer subarrays
>    // with maximum sum not exceeding maxSumAllowed
>    function check(maxSumAllowed) {
>        let subarraysNeeded = 1;
>        let currentSum = 0;
>
>        for (let num of nums) {
>            if (currentSum + num > maxSumAllowed) {
>                subarraysNeeded++;
>                currentSum = num;
>            } else {
>                currentSum += num;
>            }
>        }
>        return subarraysNeeded <= m;
>    }
>
>    // Define search space
>    let low = Math.max(...nums);
>    let high = nums.reduce((a, b) => a + b, 0);
>    let ans = high;
>
>    // Binary search for the minimum valid maximum sum
>    while (low <= high) {
>        let mid = low + Math.floor((high - low) / 2);
>
>        if (check(mid)) {
>            // mid is a possible answer. Try to find a smaller one.
>            ans = mid;
>            high = mid - 1;
>        } else {
>            // mid is too small. Need to allow a larger sum.
>            low = mid + 1;
>        }
>    }
>
>    return ans;
>}
>```

This technique turns a difficult problem into a standard binary search structure, with the main challenge being the design of the `check` function.
