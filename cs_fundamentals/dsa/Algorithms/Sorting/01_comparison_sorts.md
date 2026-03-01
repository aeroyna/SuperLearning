## Comparison-Based Sorting Algorithms

Comparison sorts are algorithms that arrange elements by comparing them to one another. The performance of these algorithms is measured by the number of comparisons they make. It has been mathematically proven that the best possible worst-case time complexity for a comparison-based sorting algorithm is **O(n log n)**.

### 1. Merge Sort
Merge Sort is a classic **Divide and Conquer** algorithm and a favorite in interviews due to its guaranteed performance and stability.

- **How it works**:
  1.  **Divide**: Recursively split the array in half until you have subarrays of size 1. A single-element array is inherently sorted.
  2.  **Conquer (Merge)**: Repeatedly merge the sorted subarrays back together. The merge step is the key: two sorted subarrays are merged into a single sorted array in O(n) time by iterating through both with two pointers and picking the smaller element at each step.
- **Time Complexity**: **O(n log n)** in all cases (best, average, and worst). The `log n` comes from the number of times the array is split, and the `n` comes from the work done at each level to merge the subarrays.
- **Space Complexity**: **O(n)**. The merge step requires temporary arrays to store the merged results, making it an out-of-place algorithm.
- **Stability**: **Stable**. It preserves the relative order of equal elements.

**Implementation:**

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>void merge(vector<int>& arr, int left, int mid, int right) {
>    int n1 = mid - left + 1;
>    int n2 = right - mid;
>
>    // Create temporary arrays
>    vector<int> leftArr(n1), rightArr(n2);
>
>    // Copy data to temporary arrays
>    for (int i = 0; i < n1; i++)
>        leftArr[i] = arr[left + i];
>    for (int j = 0; j < n2; j++)
>        rightArr[j] = arr[mid + 1 + j];
>
>    // Merge the temporary arrays back
>    int i = 0, j = 0, k = left;
>    while (i < n1 && j < n2) {
>        if (leftArr[i] <= rightArr[j]) {
>            arr[k] = leftArr[i];
>            i++;
>        } else {
>            arr[k] = rightArr[j];
>            j++;
>        }
>        k++;
>    }
>
>    // Copy remaining elements
>    while (i < n1) {
>        arr[k] = leftArr[i];
>        i++;
>        k++;
>    }
>    while (j < n2) {
>        arr[k] = rightArr[j];
>        j++;
>        k++;
>    }
>}
>
>void mergeSort(vector<int>& arr, int left, int right) {
>    if (left < right) {
>        int mid = left + (right - left) / 2;
>
>        // Sort first and second halves
>        mergeSort(arr, left, mid);
>        mergeSort(arr, mid + 1, right);
>
>        // Merge the sorted halves
>        merge(arr, left, mid, right);
>    }
>}
>
>// Usage: mergeSort(arr, 0, arr.size() - 1);
>```

>[!example]- Java
>```java
>public class MergeSort {
>    public static void merge(int[] arr, int left, int mid, int right) {
>        int n1 = mid - left + 1;
>        int n2 = right - mid;
>
>        // Create temporary arrays
>        int[] leftArr = new int[n1];
>        int[] rightArr = new int[n2];
>
>        // Copy data to temporary arrays
>        for (int i = 0; i < n1; i++)
>            leftArr[i] = arr[left + i];
>        for (int j = 0; j < n2; j++)
>            rightArr[j] = arr[mid + 1 + j];
>
>        // Merge the temporary arrays back
>        int i = 0, j = 0, k = left;
>        while (i < n1 && j < n2) {
>            if (leftArr[i] <= rightArr[j]) {
>                arr[k] = leftArr[i];
>                i++;
>            } else {
>                arr[k] = rightArr[j];
>                j++;
>            }
>            k++;
>        }
>
>        // Copy remaining elements
>        while (i < n1) {
>            arr[k] = leftArr[i];
>            i++;
>            k++;
>        }
>        while (j < n2) {
>            arr[k] = rightArr[j];
>            j++;
>            k++;
>        }
>    }
>
>    public static void mergeSort(int[] arr, int left, int right) {
>        if (left < right) {
>            int mid = left + (right - left) / 2;
>
>            // Sort first and second halves
>            mergeSort(arr, left, mid);
>            mergeSort(arr, mid + 1, right);
>
>            // Merge the sorted halves
>            merge(arr, left, mid, right);
>        }
>    }
>
>    // Usage: mergeSort(arr, 0, arr.length - 1);
>}
>```

>[!example]- Python
>```python
>def merge_sort(arr):
>    if len(arr) <= 1:
>        return arr
>
>    # Divide the array into two halves
>    mid = len(arr) // 2
>    left = merge_sort(arr[:mid])
>    right = merge_sort(arr[mid:])
>
>    # Merge the sorted halves
>    return merge(left, right)
>
>def merge(left, right):
>    result = []
>    i = j = 0
>
>    # Merge elements in sorted order
>    while i < len(left) and j < len(right):
>        if left[i] <= right[j]:
>            result.append(left[i])
>            i += 1
>        else:
>            result.append(right[j])
>            j += 1
>
>    # Add remaining elements
>    result.extend(left[i:])
>    result.extend(right[j:])
>
>    return result
>
># Usage: sorted_arr = merge_sort(arr)
>```

>[!example]- JavaScript
>```javascript
>function mergeSort(arr) {
>    if (arr.length <= 1) {
>        return arr;
>    }
>
>    // Divide the array into two halves
>    const mid = Math.floor(arr.length / 2);
>    const left = mergeSort(arr.slice(0, mid));
>    const right = mergeSort(arr.slice(mid));
>
>    // Merge the sorted halves
>    return merge(left, right);
>}
>
>function merge(left, right) {
>    const result = [];
>    let i = 0, j = 0;
>
>    // Merge elements in sorted order
>    while (i < left.length && j < right.length) {
>        if (left[i] <= right[j]) {
>            result.push(left[i]);
>            i++;
>        } else {
>            result.push(right[j]);
>            j++;
>        }
>    }
>
>    // Add remaining elements
>    return result.concat(left.slice(i)).concat(right.slice(j));
>}
>
>// Usage: const sortedArr = mergeSort(arr);
>```

### 2. Quick Sort
Quick Sort is another powerful Divide and Conquer algorithm. It is often faster than Merge Sort in practice due to better cache performance and being an in-place sort, but its worst-case performance is worse.

- **How it works**:
  1.  **Pivot**: Choose an element from the array to be the "pivot."
  2.  **Partition**: Rearrange the array so that all elements smaller than the pivot come before it, and all elements greater come after it. The pivot is now in its final sorted position. This is the core operation.
  3.  **Recurse**: Recursively apply the above steps to the subarrays of elements with smaller and larger values.
- **Time Complexity**:
  - **Average Case**: **O(n log n)**. With a good choice of pivot (e.g., a random element), the partitioning step tends to divide the array into two roughly equal halves.
  - **Worst Case**: **O(n^2)**. This occurs if the pivot is consistently chosen to be the smallest or largest element, leading to unbalanced partitions (e.g., sorting an already-sorted array with the last element as the pivot).
- **Space Complexity**: **O(log n)** on average, for the recursion call stack. It's an in-place algorithm as the partitioning happens within the original array.
- **Stability**: **Not Stable**. The partitioning process can change the relative order of equal elements.

**Implementation:**

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>int partition(vector<int>& arr, int low, int high) {
>    int pivot = arr[high];  // Choose rightmost element as pivot
>    int i = low - 1;  // Index of smaller element
>
>    for (int j = low; j < high; j++) {
>        // If current element is smaller than or equal to pivot
>        if (arr[j] <= pivot) {
>            i++;
>            swap(arr[i], arr[j]);
>        }
>    }
>    swap(arr[i + 1], arr[high]);
>    return i + 1;
>}
>
>void quickSort(vector<int>& arr, int low, int high) {
>    if (low < high) {
>        // Partition the array and get the pivot index
>        int pi = partition(arr, low, high);
>
>        // Recursively sort elements before and after partition
>        quickSort(arr, low, pi - 1);
>        quickSort(arr, pi + 1, high);
>    }
>}
>
>// Usage: quickSort(arr, 0, arr.size() - 1);
>```

>[!example]- Java
>```java
>public class QuickSort {
>    public static int partition(int[] arr, int low, int high) {
>        int pivot = arr[high];  // Choose rightmost element as pivot
>        int i = low - 1;  // Index of smaller element
>
>        for (int j = low; j < high; j++) {
>            // If current element is smaller than or equal to pivot
>            if (arr[j] <= pivot) {
>                i++;
>                // Swap arr[i] and arr[j]
>                int temp = arr[i];
>                arr[i] = arr[j];
>                arr[j] = temp;
>            }
>        }
>
>        // Swap arr[i+1] and arr[high] (pivot)
>        int temp = arr[i + 1];
>        arr[i + 1] = arr[high];
>        arr[high] = temp;
>
>        return i + 1;
>    }
>
>    public static void quickSort(int[] arr, int low, int high) {
>        if (low < high) {
>            // Partition the array and get the pivot index
>            int pi = partition(arr, low, high);
>
>            // Recursively sort elements before and after partition
>            quickSort(arr, low, pi - 1);
>            quickSort(arr, pi + 1, high);
>        }
>    }
>
>    // Usage: quickSort(arr, 0, arr.length - 1);
>}
>```

>[!example]- Python
>```python
>def quick_sort(arr, low=0, high=None):
>    if high is None:
>        high = len(arr) - 1
>
>    if low < high:
>        # Partition the array and get the pivot index
>        pi = partition(arr, low, high)
>
>        # Recursively sort elements before and after partition
>        quick_sort(arr, low, pi - 1)
>        quick_sort(arr, pi + 1, high)
>
>    return arr
>
>def partition(arr, low, high):
>    pivot = arr[high]  # Choose rightmost element as pivot
>    i = low - 1  # Index of smaller element
>
>    for j in range(low, high):
>        # If current element is smaller than or equal to pivot
>        if arr[j] <= pivot:
>            i += 1
>            arr[i], arr[j] = arr[j], arr[i]  # Swap
>
>    arr[i + 1], arr[high] = arr[high], arr[i + 1]  # Swap pivot
>    return i + 1
>
># Usage: quick_sort(arr)
>```

>[!example]- JavaScript
>```javascript
>function quickSort(arr, low = 0, high = arr.length - 1) {
>    if (low < high) {
>        // Partition the array and get the pivot index
>        const pi = partition(arr, low, high);
>
>        // Recursively sort elements before and after partition
>        quickSort(arr, low, pi - 1);
>        quickSort(arr, pi + 1, high);
>    }
>
>    return arr;
>}
>
>function partition(arr, low, high) {
>    const pivot = arr[high];  // Choose rightmost element as pivot
>    let i = low - 1;  // Index of smaller element
>
>    for (let j = low; j < high; j++) {
>        // If current element is smaller than or equal to pivot
>        if (arr[j] <= pivot) {
>            i++;
>            // Swap arr[i] and arr[j]
>            [arr[i], arr[j]] = [arr[j], arr[i]];
>        }
>    }
>
>    // Swap arr[i+1] and arr[high] (pivot)
>    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
>
>    return i + 1;
>}
>
>// Usage: quickSort(arr);
>```

### 3. Heap Sort
Heap Sort uses a Heap data structure to sort elements.
- **How it works**:
  1.  **Build Heap**: First, build a max-heap from the input array. This can be done in O(n) time (`heapify`).
  2.  **Sort**: The largest element is now at the root of the heap. Swap it with the last element of the array. The sorted portion of the array is now at the end.
  3.  "Remove" the swapped element from the heap (by reducing the heap's size by 1) and "sift down" the new root to restore the heap property.
  4.  Repeat this process until the heap is empty.
- **Time Complexity**: **O(n log n)** in all cases. Building the heap is O(n), and each of the `n` `pop` operations takes O(log n) time.
- **Space Complexity**: **O(1)**. It is an in-place sorting algorithm.
- **Stability**: **Not Stable**.

**Implementation:**

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>void heapify(vector<int>& arr, int n, int i) {
>    int largest = i;  // Initialize largest as root
>    int left = 2 * i + 1;
>    int right = 2 * i + 2;
>
>    // If left child is larger than root
>    if (left < n && arr[left] > arr[largest])
>        largest = left;
>
>    // If right child is larger than largest so far
>    if (right < n && arr[right] > arr[largest])
>        largest = right;
>
>    // If largest is not root
>    if (largest != i) {
>        swap(arr[i], arr[largest]);
>
>        // Recursively heapify the affected sub-tree
>        heapify(arr, n, largest);
>    }
>}
>
>void heapSort(vector<int>& arr) {
>    int n = arr.size();
>
>    // Build max heap
>    for (int i = n / 2 - 1; i >= 0; i--)
>        heapify(arr, n, i);
>
>    // Extract elements from heap one by one
>    for (int i = n - 1; i > 0; i--) {
>        // Move current root to end
>        swap(arr[0], arr[i]);
>
>        // Call heapify on the reduced heap
>        heapify(arr, i, 0);
>    }
>}
>
>// Usage: heapSort(arr);
>```

>[!example]- Java
>```java
>public class HeapSort {
>    public static void heapify(int[] arr, int n, int i) {
>        int largest = i;  // Initialize largest as root
>        int left = 2 * i + 1;
>        int right = 2 * i + 2;
>
>        // If left child is larger than root
>        if (left < n && arr[left] > arr[largest])
>            largest = left;
>
>        // If right child is larger than largest so far
>        if (right < n && arr[right] > arr[largest])
>            largest = right;
>
>        // If largest is not root
>        if (largest != i) {
>            int temp = arr[i];
>            arr[i] = arr[largest];
>            arr[largest] = temp;
>
>            // Recursively heapify the affected sub-tree
>            heapify(arr, n, largest);
>        }
>    }
>
>    public static void heapSort(int[] arr) {
>        int n = arr.length;
>
>        // Build max heap
>        for (int i = n / 2 - 1; i >= 0; i--)
>            heapify(arr, n, i);
>
>        // Extract elements from heap one by one
>        for (int i = n - 1; i > 0; i--) {
>            // Move current root to end
>            int temp = arr[0];
>            arr[0] = arr[i];
>            arr[i] = temp;
>
>            // Call heapify on the reduced heap
>            heapify(arr, i, 0);
>        }
>    }
>
>    // Usage: heapSort(arr);
>}
>```

>[!example]- Python
>```python
>def heapify(arr, n, i):
>    largest = i  # Initialize largest as root
>    left = 2 * i + 1
>    right = 2 * i + 2
>
>    # If left child is larger than root
>    if left < n and arr[left] > arr[largest]:
>        largest = left
>
>    # If right child is larger than largest so far
>    if right < n and arr[right] > arr[largest]:
>        largest = right
>
>    # If largest is not root
>    if largest != i:
>        arr[i], arr[largest] = arr[largest], arr[i]  # Swap
>
>        # Recursively heapify the affected sub-tree
>        heapify(arr, n, largest)
>
>def heap_sort(arr):
>    n = len(arr)
>
>    # Build max heap
>    for i in range(n // 2 - 1, -1, -1):
>        heapify(arr, n, i)
>
>    # Extract elements from heap one by one
>    for i in range(n - 1, 0, -1):
>        arr[0], arr[i] = arr[i], arr[0]  # Swap
>        heapify(arr, i, 0)
>
>    return arr
>
># Usage: heap_sort(arr)
>```

>[!example]- JavaScript
>```javascript
>function heapify(arr, n, i) {
>    let largest = i;  // Initialize largest as root
>    const left = 2 * i + 1;
>    const right = 2 * i + 2;
>
>    // If left child is larger than root
>    if (left < n && arr[left] > arr[largest])
>        largest = left;
>
>    // If right child is larger than largest so far
>    if (right < n && arr[right] > arr[largest])
>        largest = right;
>
>    // If largest is not root
>    if (largest !== i) {
>        [arr[i], arr[largest]] = [arr[largest], arr[i]];  // Swap
>
>        // Recursively heapify the affected sub-tree
>        heapify(arr, n, largest);
>    }
>}
>
>function heapSort(arr) {
>    const n = arr.length;
>
>    // Build max heap
>    for (let i = Math.floor(n / 2) - 1; i >= 0; i--)
>        heapify(arr, n, i);
>
>    // Extract elements from heap one by one
>    for (let i = n - 1; i > 0; i--) {
>        // Move current root to end
>        [arr[0], arr[i]] = [arr[i], arr[0]];  // Swap
>
>        // Call heapify on the reduced heap
>        heapify(arr, i, 0);
>    }
>
>    return arr;
>}
>
>// Usage: heapSort(arr);
>```
