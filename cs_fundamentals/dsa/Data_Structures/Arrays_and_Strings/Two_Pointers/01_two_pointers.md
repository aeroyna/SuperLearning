# Two Pointers Technique

Two pointers is an extremely common technique used to solve array and string problems. It involves using two integer variables that both move along an iterable.

## Core Concept

Instead of using nested loops O(n²), two pointers allows us to achieve O(n) time by cleverly moving two indices through the data.

## Pattern 1: Opposite Direction (Converging)

Start pointers at the edges and move them toward each other.

### Template

> [!example]- C++
> ```cpp
> void twoPointersConverging(vector<int>& arr) {
>     int left = 0;
>     int right = arr.size() - 1;
>
>     while (left < right) {
>         // Do some logic depending on the problem
>
>         // Decide which pointer to move:
>         // 1. left++
>         // 2. right--
>         // 3. Both
>     }
> }
> ```

> [!example]- Java
> ```java
> void twoPointersConverging(int[] arr) {
>     int left = 0;
>     int right = arr.length - 1;
>
>     while (left < right) {
>         // Do some logic depending on the problem
>
>         // Decide which pointer to move:
>         // 1. left++
>         // 2. right--
>         // 3. Both
>     }
> }
> ```

> [!example]- Python
> ```python
> def two_pointers_converging(arr):
>     left = 0
>     right = len(arr) - 1
>
>     while left < right:
>         # Do some logic depending on the problem
>
>         # Decide which pointer to move:
>         # 1. left += 1
>         # 2. right -= 1
>         # 3. Both
>         pass
> ```

> [!example]- JavaScript
> ```javascript
> function twoPointersConverging(arr) {
>     let left = 0;
>     let right = arr.length - 1;
>
>     while (left < right) {
>         // Do some logic depending on the problem
>
>         // Decide which pointer to move:
>         // 1. left++
>         // 2. right--
>         // 3. Both
>     }
> }
> ```

### Why O(n)?

The pointers start `n` positions apart and move at least one step closer in each iteration. Maximum iterations = n.

### Example: Check Palindrome

> [!example]- C++
> ```cpp
> bool isPalindrome(string s) {
>     int left = 0, right = s.length() - 1;
>
>     while (left < right) {
>         if (s[left] != s[right]) {
>             return false;
>         }
>         left++;
>         right--;
>     }
>     return true;
> }
> ```

> [!example]- Java
> ```java
> boolean isPalindrome(String s) {
>     int left = 0, right = s.length() - 1;
>
>     while (left < right) {
>         if (s.charAt(left) != s.charAt(right)) {
>             return false;
>         }
>         left++;
>         right--;
>     }
>     return true;
> }
> ```

> [!example]- Python
> ```python
> def isPalindrome(s):
>     left, right = 0, len(s) - 1
>
>     while left < right:
>         if s[left] != s[right]:
>             return False
>         left += 1
>         right -= 1
>
>     return True
> ```

> [!example]- JavaScript
> ```javascript
> function isPalindrome(s) {
>     let left = 0, right = s.length - 1;
>
>     while (left < right) {
>         if (s[left] !== s[right]) {
>             return false;
>         }
>         left++;
>         right--;
>     }
>     return true;
> }
> ```

### Example: Two Sum (Sorted Array)

> [!example]- C++
> ```cpp
> vector<int> twoSum(vector<int>& numbers, int target) {
>     int left = 0, right = numbers.size() - 1;
>
>     while (left < right) {
>         int sum = numbers[left] + numbers[right];
>
>         if (sum == target) {
>             return {left + 1, right + 1}; // 1-indexed
>         } else if (sum < target) {
>             left++;
>         } else {
>             right--;
>         }
>     }
>     return {};
> }
> ```

> [!example]- Java
> ```java
> int[] twoSum(int[] numbers, int target) {
>     int left = 0, right = numbers.length - 1;
>
>     while (left < right) {
>         int sum = numbers[left] + numbers[right];
>
>         if (sum == target) {
>             return new int[]{left + 1, right + 1}; // 1-indexed
>         } else if (sum < target) {
>             left++;
>         } else {
>             right--;
>         }
>     }
>     return new int[]{};
> }
> ```

> [!example]- Python
> ```python
> def twoSum(numbers, target):
>     left, right = 0, len(numbers) - 1
>
>     while left < right:
>         current_sum = numbers[left] + numbers[right]
>
>         if current_sum == target:
>             return [left + 1, right + 1]  # 1-indexed
>         elif current_sum < target:
>             left += 1
>         else:
>             right -= 1
>
>     return []
> ```

> [!example]- JavaScript
> ```javascript
> function twoSum(numbers, target) {
>     let left = 0, right = numbers.length - 1;
>
>     while (left < right) {
>         const sum = numbers[left] + numbers[right];
>
>         if (sum === target) {
>             return [left + 1, right + 1]; // 1-indexed
>         } else if (sum < target) {
>             left++;
>         } else {
>             right--;
>         }
>     }
>     return [];
> }
> ```

### Example: Container With Most Water

> [!example]- C++
> ```cpp
> int maxArea(vector<int>& height) {
>     int left = 0, right = height.size() - 1;
>     int maxWater = 0;
>
>     while (left < right) {
>         int width = right - left;
>         int h = min(height[left], height[right]);
>         maxWater = max(maxWater, width * h);
>
>         if (height[left] < height[right]) {
>             left++;
>         } else {
>             right--;
>         }
>     }
>     return maxWater;
> }
> ```

> [!example]- Java
> ```java
> int maxArea(int[] height) {
>     int left = 0, right = height.length - 1;
>     int maxWater = 0;
>
>     while (left < right) {
>         int width = right - left;
>         int h = Math.min(height[left], height[right]);
>         maxWater = Math.max(maxWater, width * h);
>
>         if (height[left] < height[right]) {
>             left++;
>         } else {
>             right--;
>         }
>     }
>     return maxWater;
> }
> ```

> [!example]- Python
> ```python
> def maxArea(height):
>     left, right = 0, len(height) - 1
>     max_water = 0
>
>     while left < right:
>         width = right - left
>         h = min(height[left], height[right])
>         max_water = max(max_water, width * h)
>
>         if height[left] < height[right]:
>             left += 1
>         else:
>             right -= 1
>
>     return max_water
> ```

> [!example]- JavaScript
> ```javascript
> function maxArea(height) {
>     let left = 0, right = height.length - 1;
>     let maxWater = 0;
>
>     while (left < right) {
>         const width = right - left;
>         const h = Math.min(height[left], height[right]);
>         maxWater = Math.max(maxWater, width * h);
>
>         if (height[left] < height[right]) {
>             left++;
>         } else {
>             right--;
>         }
>     }
>     return maxWater;
> }
> ```

## Pattern 2: Same Direction (Two Arrays)

Both pointers start at beginning, move through two different arrays.

### Template

> [!example]- C++
> ```cpp
> void twoPointersTwoArrays(vector<int>& arr1, vector<int>& arr2) {
>     int i = 0, j = 0;
>
>     while (i < arr1.size() && j < arr2.size()) {
>         // Do some logic depending on the problem
>         // Decide which pointer to move
>     }
>
>     // Handle remaining elements
>     while (i < arr1.size()) i++;
>     while (j < arr2.size()) j++;
> }
> ```

> [!example]- Java
> ```java
> void twoPointersTwoArrays(int[] arr1, int[] arr2) {
>     int i = 0, j = 0;
>
>     while (i < arr1.length && j < arr2.length) {
>         // Do some logic depending on the problem
>         // Decide which pointer to move
>     }
>
>     // Handle remaining elements
>     while (i < arr1.length) i++;
>     while (j < arr2.length) j++;
> }
> ```

> [!example]- Python
> ```python
> def two_pointers_two_arrays(arr1, arr2):
>     i = j = 0
>
>     while i < len(arr1) and j < len(arr2):
>         # Do some logic depending on the problem
>         # Decide which pointer to move
>         pass
>
>     # Handle remaining elements
>     while i < len(arr1):
>         i += 1
>     while j < len(arr2):
>         j += 1
> ```

> [!example]- JavaScript
> ```javascript
> function twoPointersTwoArrays(arr1, arr2) {
>     let i = 0, j = 0;
>
>     while (i < arr1.length && j < arr2.length) {
>         // Do some logic depending on the problem
>         // Decide which pointer to move
>     }
>
>     // Handle remaining elements
>     while (i < arr1.length) i++;
>     while (j < arr2.length) j++;
> }
> ```

### Example: Merge Two Sorted Arrays

> [!example]- C++
> ```cpp
> void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
>     int p1 = m - 1, p2 = n - 1, p = m + n - 1;
>
>     while (p1 >= 0 && p2 >= 0) {
>         if (nums1[p1] > nums2[p2]) {
>             nums1[p] = nums1[p1--];
>         } else {
>             nums1[p] = nums2[p2--];
>         }
>         p--;
>     }
>
>     while (p2 >= 0) {
>         nums1[p--] = nums2[p2--];
>     }
> }
> ```

> [!example]- Java
> ```java
> void merge(int[] nums1, int m, int[] nums2, int n) {
>     int p1 = m - 1, p2 = n - 1, p = m + n - 1;
>
>     while (p1 >= 0 && p2 >= 0) {
>         if (nums1[p1] > nums2[p2]) {
>             nums1[p] = nums1[p1--];
>         } else {
>             nums1[p] = nums2[p2--];
>         }
>         p--;
>     }
>
>     while (p2 >= 0) {
>         nums1[p--] = nums2[p2--];
>     }
> }
> ```

> [!example]- Python
> ```python
> def merge(nums1, m, nums2, n):
>     p1, p2, p = m - 1, n - 1, m + n - 1
>
>     while p1 >= 0 and p2 >= 0:
>         if nums1[p1] > nums2[p2]:
>             nums1[p] = nums1[p1]
>             p1 -= 1
>         else:
>             nums1[p] = nums2[p2]
>             p2 -= 1
>         p -= 1
>
>     while p2 >= 0:
>         nums1[p] = nums2[p2]
>         p2 -= 1
>         p -= 1
> ```

> [!example]- JavaScript
> ```javascript
> function merge(nums1, m, nums2, n) {
>     let p1 = m - 1, p2 = n - 1, p = m + n - 1;
>
>     while (p1 >= 0 && p2 >= 0) {
>         if (nums1[p1] > nums2[p2]) {
>             nums1[p] = nums1[p1--];
>         } else {
>             nums1[p] = nums2[p2--];
>         }
>         p--;
>     }
>
>     while (p2 >= 0) {
>         nums1[p--] = nums2[p2--];
>     }
> }
> ```

### Example: Is Subsequence

> [!example]- C++
> ```cpp
> bool isSubsequence(string s, string t) {
>     int i = 0, j = 0;
>
>     while (i < s.length() && j < t.length()) {
>         if (s[i] == t[j]) {
>             i++;
>         }
>         j++;
>     }
>
>     return i == s.length();
> }
> ```

> [!example]- Java
> ```java
> boolean isSubsequence(String s, String t) {
>     int i = 0, j = 0;
>
>     while (i < s.length() && j < t.length()) {
>         if (s.charAt(i) == t.charAt(j)) {
>             i++;
>         }
>         j++;
>     }
>
>     return i == s.length();
> }
> ```

> [!example]- Python
> ```python
> def isSubsequence(s, t):
>     i = j = 0
>
>     while i < len(s) and j < len(t):
>         if s[i] == t[j]:
>             i += 1
>         j += 1
>
>     return i == len(s)
> ```

> [!example]- JavaScript
> ```javascript
> function isSubsequence(s, t) {
>     let i = 0, j = 0;
>
>     while (i < s.length && j < t.length) {
>         if (s[i] === t[j]) {
>             i++;
>         }
>         j++;
>     }
>
>     return i === s.length;
> }
> ```

## Pattern 3: Fast and Slow Pointers

One pointer moves faster than the other (commonly in linked lists, but applicable to arrays).

### Example: Remove Duplicates (Sorted Array)

> [!example]- C++
> ```cpp
> int removeDuplicates(vector<int>& nums) {
>     if (nums.empty()) return 0;
>
>     int slow = 0;
>     for (int fast = 1; fast < nums.size(); fast++) {
>         if (nums[fast] != nums[slow]) {
>             slow++;
>             nums[slow] = nums[fast];
>         }
>     }
>     return slow + 1;
> }
> ```

> [!example]- Java
> ```java
> int removeDuplicates(int[] nums) {
>     if (nums.length == 0) return 0;
>
>     int slow = 0;
>     for (int fast = 1; fast < nums.length; fast++) {
>         if (nums[fast] != nums[slow]) {
>             slow++;
>             nums[slow] = nums[fast];
>         }
>     }
>     return slow + 1;
> }
> ```

> [!example]- Python
> ```python
> def removeDuplicates(nums):
>     if not nums:
>         return 0
>
>     slow = 0
>     for fast in range(1, len(nums)):
>         if nums[fast] != nums[slow]:
>             slow += 1
>             nums[slow] = nums[fast]
>
>     return slow + 1
> ```

> [!example]- JavaScript
> ```javascript
> function removeDuplicates(nums) {
>     if (nums.length === 0) return 0;
>
>     let slow = 0;
>     for (let fast = 1; fast < nums.length; fast++) {
>         if (nums[fast] !== nums[slow]) {
>             slow++;
>             nums[slow] = nums[fast];
>         }
>     }
>     return slow + 1;
> }
> ```

### Example: Move Zeroes

> [!example]- C++
> ```cpp
> void moveZeroes(vector<int>& nums) {
>     int slow = 0;
>
>     for (int fast = 0; fast < nums.size(); fast++) {
>         if (nums[fast] != 0) {
>             swap(nums[slow], nums[fast]);
>             slow++;
>         }
>     }
> }
> ```

> [!example]- Java
> ```java
> void moveZeroes(int[] nums) {
>     int slow = 0;
>
>     for (int fast = 0; fast < nums.length; fast++) {
>         if (nums[fast] != 0) {
>             int temp = nums[slow];
>             nums[slow] = nums[fast];
>             nums[fast] = temp;
>             slow++;
>         }
>     }
> }
> ```

> [!example]- Python
> ```python
> def moveZeroes(nums):
>     slow = 0
>
>     for fast in range(len(nums)):
>         if nums[fast] != 0:
>             nums[slow], nums[fast] = nums[fast], nums[slow]
>             slow += 1
> ```

> [!example]- JavaScript
> ```javascript
> function moveZeroes(nums) {
>     let slow = 0;
>
>     for (let fast = 0; fast < nums.length; fast++) {
>         if (nums[fast] !== 0) {
>             [nums[slow], nums[fast]] = [nums[fast], nums[slow]];
>             slow++;
>         }
>     }
> }
> ```

## Pattern 4: Three Pointers

Extension of two pointers for problems like 3Sum.

### Example: Three Sum

> [!example]- C++
> ```cpp
> vector<vector<int>> threeSum(vector<int>& nums) {
>     sort(nums.begin(), nums.end());
>     vector<vector<int>> result;
>
>     for (int i = 0; i < nums.size() - 2; i++) {
>         if (i > 0 && nums[i] == nums[i - 1]) continue;
>
>         int left = i + 1, right = nums.size() - 1;
>
>         while (left < right) {
>             int sum = nums[i] + nums[left] + nums[right];
>
>             if (sum == 0) {
>                 result.push_back({nums[i], nums[left], nums[right]});
>                 while (left < right && nums[left] == nums[left + 1]) left++;
>                 while (left < right && nums[right] == nums[right - 1]) right--;
>                 left++;
>                 right--;
>             } else if (sum < 0) {
>                 left++;
>             } else {
>                 right--;
>             }
>         }
>     }
>     return result;
> }
> ```

> [!example]- Java
> ```java
> List<List<Integer>> threeSum(int[] nums) {
>     Arrays.sort(nums);
>     List<List<Integer>> result = new ArrayList<>();
>
>     for (int i = 0; i < nums.length - 2; i++) {
>         if (i > 0 && nums[i] == nums[i - 1]) continue;
>
>         int left = i + 1, right = nums.length - 1;
>
>         while (left < right) {
>             int sum = nums[i] + nums[left] + nums[right];
>
>             if (sum == 0) {
>                 result.add(Arrays.asList(nums[i], nums[left], nums[right]));
>                 while (left < right && nums[left] == nums[left + 1]) left++;
>                 while (left < right && nums[right] == nums[right - 1]) right--;
>                 left++;
>                 right--;
>             } else if (sum < 0) {
>                 left++;
>             } else {
>                 right--;
>             }
>         }
>     }
>     return result;
> }
> ```

> [!example]- Python
> ```python
> def threeSum(nums):
>     nums.sort()
>     result = []
>
>     for i in range(len(nums) - 2):
>         if i > 0 and nums[i] == nums[i - 1]:
>             continue
>
>         left, right = i + 1, len(nums) - 1
>
>         while left < right:
>             total = nums[i] + nums[left] + nums[right]
>
>             if total == 0:
>                 result.append([nums[i], nums[left], nums[right]])
>                 while left < right and nums[left] == nums[left + 1]:
>                     left += 1
>                 while left < right and nums[right] == nums[right - 1]:
>                     right -= 1
>                 left += 1
>                 right -= 1
>             elif total < 0:
>                 left += 1
>             else:
>                 right -= 1
>
>     return result
> ```

> [!example]- JavaScript
> ```javascript
> function threeSum(nums) {
>     nums.sort((a, b) => a - b);
>     const result = [];
>
>     for (let i = 0; i < nums.length - 2; i++) {
>         if (i > 0 && nums[i] === nums[i - 1]) continue;
>
>         let left = i + 1, right = nums.length - 1;
>
>         while (left < right) {
>             const sum = nums[i] + nums[left] + nums[right];
>
>             if (sum === 0) {
>                 result.push([nums[i], nums[left], nums[right]]);
>                 while (left < right && nums[left] === nums[left + 1]) left++;
>                 while (left < right && nums[right] === nums[right - 1]) right--;
>                 left++;
>                 right--;
>             } else if (sum < 0) {
>                 left++;
>             } else {
>                 right--;
>             }
>         }
>     }
>     return result;
> }
> ```

## When to Use Two Pointers

1. **Sorted arrays** - Many two pointer problems require sorting
2. **Finding pairs** - Two elements that satisfy some condition
3. **Comparing from both ends** - Palindrome, container problems
4. **Merging sorted data** - Merge sort, merge intervals
5. **Partitioning** - Dutch flag, move zeros

## Common Mistakes

1. **Off-by-one errors** in loop conditions
2. **Forgetting to handle duplicates**
3. **Not considering empty or single-element arrays**
4. **Incorrect pointer movement logic**

## Practice Problems

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Valid Palindrome | Converging | Easy |
| Two Sum II | Converging | Medium |
| 3Sum | Three Pointers | Medium |
| Container With Most Water | Converging | Medium |
| Remove Duplicates | Fast-Slow | Easy |
| Move Zeroes | Fast-Slow | Easy |
| Merge Sorted Array | Two Arrays | Easy |
| Is Subsequence | Two Arrays | Easy |
| Sort Colors | Three Pointers | Medium |
| Trapping Rain Water | Converging | Hard |
