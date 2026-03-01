## Non-Comparison Sorting Algorithms

While comparison sorts have a theoretical lower bound of O(n log n), non-comparison sorts can achieve faster runtimes, often **linear time O(n)**, by making assumptions about the data being sorted. They don't compare elements to each other but instead use other properties, like their digit values or their integer range, to determine the final sorted order.

These algorithms are less general-purpose than comparison sorts but are extremely fast for the specific types of data they are designed for.

### 1. Counting Sort
Counting Sort is an efficient algorithm for sorting a collection of items that have a known, small integer key range.

- **How it works**:
  1.  Find the maximum element (`max_val`) in the input array to determine the range of values.
  2.  Create a `count` array of size `max_val + 1`, initialized to all zeros.
  3.  Iterate through the input array. For each element, increment the corresponding index in the `count` array. (e.g., if you see the number `3` five times, `count[3]` will be `5`).
  4.  The `count` array now holds the frequency of each element.
  5.  Reconstruct the sorted array by iterating through the `count` array. For each number `i` from 0 to `max_val`, add `i` to the result array `count[i]` times.
- **Time Complexity**: **O(n + k)**, where `n` is the number of elements and `k` is the range of the input values (the `max_val`). This simplifies to O(n) if `k` is on the order of `n`.
- **Space Complexity**: **O(k)** to store the count array.
- **When to use**: Excellent when `k` is not significantly larger than `n`. For example, sorting an array of ages (where the range is likely 0-150) or lowercase characters. It becomes impractical if the range is very large (e.g., sorting 32-bit integers).

**Implementation:**

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>using namespace std;
>
>vector<int> countingSort(vector<int>& arr) {
>    if (arr.empty()) return arr;
>
>    // Find the maximum element
>    int max_val = *max_element(arr.begin(), arr.end());
>    int min_val = *min_element(arr.begin(), arr.end());
>    int range = max_val - min_val + 1;
>
>    // Create count array
>    vector<int> count(range, 0);
>    vector<int> output(arr.size());
>
>    // Store count of each element
>    for (int i = 0; i < arr.size(); i++)
>        count[arr[i] - min_val]++;
>
>    // Change count[i] so it contains actual position
>    for (int i = 1; i < range; i++)
>        count[i] += count[i - 1];
>
>    // Build the output array
>    for (int i = arr.size() - 1; i >= 0; i--) {
>        output[count[arr[i] - min_val] - 1] = arr[i];
>        count[arr[i] - min_val]--;
>    }
>
>    return output;
>}
>
>// Usage: vector<int> sorted = countingSort(arr);
>```

>[!example]- Java
>```java
>import java.util.Arrays;
>
>public class CountingSort {
>    public static int[] countingSort(int[] arr) {
>        if (arr.length == 0) return arr;
>
>        // Find the maximum and minimum elements
>        int max_val = Arrays.stream(arr).max().getAsInt();
>        int min_val = Arrays.stream(arr).min().getAsInt();
>        int range = max_val - min_val + 1;
>
>        // Create count array
>        int[] count = new int[range];
>        int[] output = new int[arr.length];
>
>        // Store count of each element
>        for (int i = 0; i < arr.length; i++)
>            count[arr[i] - min_val]++;
>
>        // Change count[i] so it contains actual position
>        for (int i = 1; i < range; i++)
>            count[i] += count[i - 1];
>
>        // Build the output array
>        for (int i = arr.length - 1; i >= 0; i--) {
>            output[count[arr[i] - min_val] - 1] = arr[i];
>            count[arr[i] - min_val]--;
>        }
>
>        return output;
>    }
>
>    // Usage: int[] sorted = countingSort(arr);
>}
>```

>[!example]- Python
>```python
>def counting_sort(arr):
>    if not arr:
>        return arr
>
>    # Find the maximum and minimum elements
>    max_val = max(arr)
>    min_val = min(arr)
>    range_size = max_val - min_val + 1
>
>    # Create count array
>    count = [0] * range_size
>    output = [0] * len(arr)
>
>    # Store count of each element
>    for num in arr:
>        count[num - min_val] += 1
>
>    # Change count[i] so it contains actual position
>    for i in range(1, range_size):
>        count[i] += count[i - 1]
>
>    # Build the output array
>    for i in range(len(arr) - 1, -1, -1):
>        output[count[arr[i] - min_val] - 1] = arr[i]
>        count[arr[i] - min_val] -= 1
>
>    return output
>
># Usage: sorted_arr = counting_sort(arr)
>```

>[!example]- JavaScript
>```javascript
>function countingSort(arr) {
>    if (arr.length === 0) return arr;
>
>    // Find the maximum and minimum elements
>    const max_val = Math.max(...arr);
>    const min_val = Math.min(...arr);
>    const range = max_val - min_val + 1;
>
>    // Create count array
>    const count = new Array(range).fill(0);
>    const output = new Array(arr.length);
>
>    // Store count of each element
>    for (let i = 0; i < arr.length; i++)
>        count[arr[i] - min_val]++;
>
>    // Change count[i] so it contains actual position
>    for (let i = 1; i < range; i++)
>        count[i] += count[i - 1];
>
>    // Build the output array
>    for (let i = arr.length - 1; i >= 0; i--) {
>        output[count[arr[i] - min_val] - 1] = arr[i];
>        count[arr[i] - min_val]--;
>    }
>
>    return output;
>}
>
>// Usage: const sorted = countingSort(arr);
>```

### 2. Radix Sort
Radix Sort is a clever algorithm that sorts integers by processing individual digits. It sorts the numbers digit by digit, from the least significant digit to the most significant digit. It uses a stable sorting algorithm (like Counting Sort) as a subroutine to sort the digits at each position.

- **How it works**:
  1.  Find the maximum number in the array to determine the number of digits to process.
  2.  For each digit position (from the 1s place, then 10s place, then 100s place, etc.):
      a. Sort the entire array of numbers **stably** based on the value of the current digit. Counting Sort is a perfect choice for this step, as digits are always in a small range (0-9).
  3.  After sorting by the most significant digit, the entire array will be sorted.
- **Time Complexity**: **O(d * (n + b))**, where `d` is the number of digits in the maximum number, `n` is the number of elements, and `b` is the base of the numbers (usually 10 for decimal or 2 for binary). If `d` is constant, the time complexity is linear, O(n).
- **Space Complexity**: **O(n + b)**, primarily from the underlying Counting Sort.
- **When to use**: Very efficient for sorting large integers. Unlike Counting Sort, its space complexity does not depend on the range of the numbers, making it more suitable for larger value ranges.

**Implementation:**

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>using namespace std;
>
>// A utility function to get the digit at a specific position
>int getDigit(int num, int exp) {
>    return (num / exp) % 10;
>}
>
>// Counting sort for a specific digit position
>void countingSortByDigit(vector<int>& arr, int exp) {
>    int n = arr.size();
>    vector<int> output(n);
>    vector<int> count(10, 0);
>
>    // Store count of occurrences
>    for (int i = 0; i < n; i++)
>        count[getDigit(arr[i], exp)]++;
>
>    // Change count[i] so it contains actual position
>    for (int i = 1; i < 10; i++)
>        count[i] += count[i - 1];
>
>    // Build the output array
>    for (int i = n - 1; i >= 0; i--) {
>        int digit = getDigit(arr[i], exp);
>        output[count[digit] - 1] = arr[i];
>        count[digit]--;
>    }
>
>    // Copy the output array to arr
>    for (int i = 0; i < n; i++)
>        arr[i] = output[i];
>}
>
>void radixSort(vector<int>& arr) {
>    if (arr.empty()) return;
>
>    // Find the maximum number to know the number of digits
>    int max_val = *max_element(arr.begin(), arr.end());
>
>    // Do counting sort for every digit
>    // exp is 10^i where i is the current digit position
>    for (int exp = 1; max_val / exp > 0; exp *= 10)
>        countingSortByDigit(arr, exp);
>}
>
>// Usage: radixSort(arr);
>```

>[!example]- Java
>```java
>import java.util.Arrays;
>
>public class RadixSort {
>    // A utility function to get the digit at a specific position
>    private static int getDigit(int num, int exp) {
>        return (num / exp) % 10;
>    }
>
>    // Counting sort for a specific digit position
>    private static void countingSortByDigit(int[] arr, int exp) {
>        int n = arr.length;
>        int[] output = new int[n];
>        int[] count = new int[10];
>
>        // Store count of occurrences
>        for (int i = 0; i < n; i++)
>            count[getDigit(arr[i], exp)]++;
>
>        // Change count[i] so it contains actual position
>        for (int i = 1; i < 10; i++)
>            count[i] += count[i - 1];
>
>        // Build the output array
>        for (int i = n - 1; i >= 0; i--) {
>            int digit = getDigit(arr[i], exp);
>            output[count[digit] - 1] = arr[i];
>            count[digit]--;
>        }
>
>        // Copy the output array to arr
>        for (int i = 0; i < n; i++)
>            arr[i] = output[i];
>    }
>
>    public static void radixSort(int[] arr) {
>        if (arr.length == 0) return;
>
>        // Find the maximum number to know the number of digits
>        int max_val = Arrays.stream(arr).max().getAsInt();
>
>        // Do counting sort for every digit
>        // exp is 10^i where i is the current digit position
>        for (int exp = 1; max_val / exp > 0; exp *= 10)
>            countingSortByDigit(arr, exp);
>    }
>
>    // Usage: radixSort(arr);
>}
>```

>[!example]- Python
>```python
>def counting_sort_by_digit(arr, exp):
>    """Counting sort for a specific digit position"""
>    n = len(arr)
>    output = [0] * n
>    count = [0] * 10
>
>    # Store count of occurrences
>    for i in range(n):
>        digit = (arr[i] // exp) % 10
>        count[digit] += 1
>
>    # Change count[i] so it contains actual position
>    for i in range(1, 10):
>        count[i] += count[i - 1]
>
>    # Build the output array
>    for i in range(n - 1, -1, -1):
>        digit = (arr[i] // exp) % 10
>        output[count[digit] - 1] = arr[i]
>        count[digit] -= 1
>
>    # Copy the output array to arr
>    for i in range(n):
>        arr[i] = output[i]
>
>def radix_sort(arr):
>    if not arr:
>        return arr
>
>    # Find the maximum number to know the number of digits
>    max_val = max(arr)
>
>    # Do counting sort for every digit
>    # exp is 10^i where i is the current digit position
>    exp = 1
>    while max_val // exp > 0:
>        counting_sort_by_digit(arr, exp)
>        exp *= 10
>
>    return arr
>
># Usage: radix_sort(arr)
>```

>[!example]- JavaScript
>```javascript
>// A utility function to get the digit at a specific position
>function getDigit(num, exp) {
>    return Math.floor((num / exp) % 10);
>}
>
>// Counting sort for a specific digit position
>function countingSortByDigit(arr, exp) {
>    const n = arr.length;
>    const output = new Array(n);
>    const count = new Array(10).fill(0);
>
>    // Store count of occurrences
>    for (let i = 0; i < n; i++)
>        count[getDigit(arr[i], exp)]++;
>
>    // Change count[i] so it contains actual position
>    for (let i = 1; i < 10; i++)
>        count[i] += count[i - 1];
>
>    // Build the output array
>    for (let i = n - 1; i >= 0; i--) {
>        const digit = getDigit(arr[i], exp);
>        output[count[digit] - 1] = arr[i];
>        count[digit]--;
>    }
>
>    // Copy the output array to arr
>    for (let i = 0; i < n; i++)
>        arr[i] = output[i];
>}
>
>function radixSort(arr) {
>    if (arr.length === 0) return arr;
>
>    // Find the maximum number to know the number of digits
>    const max_val = Math.max(...arr);
>
>    // Do counting sort for every digit
>    // exp is 10^i where i is the current digit position
>    for (let exp = 1; Math.floor(max_val / exp) > 0; exp *= 10)
>        countingSortByDigit(arr, exp);
>
>    return arr;
>}
>
>// Usage: radixSort(arr);
>```
