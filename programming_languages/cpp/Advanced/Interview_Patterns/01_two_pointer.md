# Two Pointer Technique

The two pointer technique uses two pointers to traverse a data structure, often from different directions or at different speeds.

## Types of Two Pointer

### 1. Opposite Direction (Start and End)

Used for: sorted arrays, palindromes, two sum variants

```cpp
// Two Sum in Sorted Array
std::vector<int> twoSum(std::vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;

    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            return {left, right};
        } else if (sum < target) {
            ++left;   // Need larger sum
        } else {
            --right;  // Need smaller sum
        }
    }
    return {};  // Not found
}
```

### 2. Same Direction (Slow and Fast)

Used for: removing duplicates, linked list cycles, partition

```cpp
// Remove Duplicates from Sorted Array
int removeDuplicates(std::vector<int>& nums) {
    if (nums.empty()) return 0;

    int slow = 0;  // Position to write
    for (int fast = 1; fast < nums.size(); ++fast) {
        if (nums[fast] != nums[slow]) {
            ++slow;
            nums[slow] = nums[fast];
        }
    }
    return slow + 1;  // New length
}
```

### 3. Multiple Arrays

Used for: merging, comparing sequences

```cpp
// Merge Two Sorted Arrays
std::vector<int> merge(std::vector<int>& a, std::vector<int>& b) {
    std::vector<int> result;
    int i = 0, j = 0;

    while (i < a.size() && j < b.size()) {
        if (a[i] <= b[j]) {
            result.push_back(a[i++]);
        } else {
            result.push_back(b[j++]);
        }
    }

    while (i < a.size()) result.push_back(a[i++]);
    while (j < b.size()) result.push_back(b[j++]);

    return result;
}
```

## Classic Problems

### Valid Palindrome

```cpp
bool isPalindrome(const std::string& s) {
    int left = 0, right = s.size() - 1;

    while (left < right) {
        // Skip non-alphanumeric
        while (left < right && !isalnum(s[left])) ++left;
        while (left < right && !isalnum(s[right])) --right;

        if (tolower(s[left]) != tolower(s[right])) {
            return false;
        }
        ++left;
        --right;
    }
    return true;
}
```

### Container With Most Water

```cpp
int maxArea(std::vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int maxWater = 0;

    while (left < right) {
        int h = std::min(height[left], height[right]);
        int w = right - left;
        maxWater = std::max(maxWater, h * w);

        // Move the shorter line inward
        if (height[left] < height[right]) {
            ++left;
        } else {
            --right;
        }
    }
    return maxWater;
}
```

### Three Sum

```cpp
std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
    std::vector<std::vector<int>> result;
    std::sort(nums.begin(), nums.end());

    for (int i = 0; i < nums.size(); ++i) {
        if (i > 0 && nums[i] == nums[i-1]) continue;  // Skip duplicates

        int left = i + 1, right = nums.size() - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.push_back({nums[i], nums[left], nums[right]});
                while (left < right && nums[left] == nums[left+1]) ++left;
                while (left < right && nums[right] == nums[right-1]) --right;
                ++left;
                --right;
            } else if (sum < 0) {
                ++left;
            } else {
                --right;
            }
        }
    }
    return result;
}
```

### Linked List Cycle Detection (Floyd's Algorithm)

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

## When to Use Two Pointer

- Array/string is **sorted**
- Need to find **pairs** with certain property
- Need to **compare/merge** elements from different positions
- Need to **partition** or **rearrange** in place
- **Palindrome** or **symmetry** checking

## Key Takeaways

- Two pointers reduce O(n²) to O(n) for many problems
- Opposite direction: sorted arrays, target sum
- Same direction: in-place modification, fast/slow
- Multiple arrays: merging, comparing
- Often combined with sorting as preprocessing

## Common Interview Questions

> [!question]- How do you know when to use two pointer?
> When dealing with sorted arrays/strings, finding pairs, or problems involving "meeting in the middle" logic. Keywords: sorted, pairs, palindrome, in-place.

> [!question]- What's the time complexity benefit?
> Often reduces O(n²) brute force to O(n), as each pointer only moves forward (or backward) through the array once.
