# Arrays: Beyond the Basics

Arrays are fundamental data structures in Java, providing a fixed-size, sequential collection of elements of the same type. While seemingly simple, their object-oriented nature and how they interact with memory have important nuances.

## 1. Arrays as Objects
Unlike C/C++, where arrays are merely pointers to a contiguous block of memory, in Java, arrays are **objects**.
*   When you create an array (`new int[5]`), you are creating an instance of `java.lang.Object` (specifically, an array class).
*   This array object exists on the **Heap**.
*   The array object inherently knows its own `length` (e.g., `myArray.length`). This is a final field of the array object.
*   Array elements, whether primitives or object references, are stored contiguously within this Heap object.

## 2. Declaration and Creation: A Deeper Look

### Syntax
```java
// Declaration:
int[] numbers;      // Preferred: type followed by []
int numbers2[];     // Valid, but less readable (C-style)

// Creation (Allocation):
numbers = new int[5]; // Allocates space for 5 integers on the Heap
```

### Default Initialization
*   When an array is created, its elements are automatically initialized to their **default values**:
    *   `0` for numeric primitives (`byte`, `short`, `int`, `long`, `float`, `double`).
    *   `\u0000` (null character) for `char`.
    *   `false` for `boolean`.
    *   `null` for object reference types.

### Initialization with Values
```java
int[] primes = {2, 3, 5, 7, 11}; // Declares, creates, and initializes
// Internally equivalent to:
// int[] primes = new int[5];
// primes[0] = 2; primes[1] = 3; ...
```

## 3. Accessing Elements and `ArrayIndexOutOfBoundsException`
*   Elements are accessed using a zero-based index: `myArray[0]` is the first element.
*   The last element is at `myArray[myArray.length - 1]`.
*   **Runtime Check:** Java performs automatic bounds checking. Accessing an index outside `0` to `length - 1` (e.g., `-1` or `length`) will throw a **`ArrayIndexOutOfBoundsException`** at runtime. This prevents memory corruption and is a key safety feature compared to C/C++.

## 4. Multi-Dimensional Arrays: Arrays of Arrays

Java does not have true multi-dimensional arrays in the mathematical sense (a single contiguous block of memory for a matrix). Instead, it has **arrays of arrays**.

### Internal Structure
A `int[][] matrix` is an array of `int[]` objects. Each `int[]` object is a separate array in the Heap.

```text
       matrix (reference on Stack)
          |
          V
+-----------------+ (Array Object on Heap)
|  [0] -> ref to +----+ (Array Object on Heap)
|  [1] -> ref to +---------+ (Array Object on Heap)
|  [2] -> ref to +------------+ (Array Object on Heap)
+-----------------+         |            |
                            V            V
                          [int, int, int] [int, int, int]
```

### Jagged Arrays (Non-Rectangular)
Because each row is an independent array, they can have different lengths.
```java
int[][] jaggedArray = new int[3][]; // Declare 3 rows, but don't define column count yet
jaggedArray[0] = new int[2];        // First row has 2 columns
jaggedArray[1] = new int[5];        // Second row has 5 columns
jaggedArray[2] = new int[1];        // Third row has 1 column
```

## 5. The `java.util.Arrays` Utility Class: Beyond Basic Manipulation

This class provides static utility methods for common array operations, enhancing their usability.

*   `Arrays.toString(arr)`: Returns a string representation of the array (e.g., `"[1, 2, 3]"`). Essential for debugging.
*   `Arrays.deepToString(matrix)`: For multi-dimensional arrays, recursively prints contents.
*   `Arrays.sort(arr)`: Sorts the array.
    *   For primitives: Uses a tuned **Dual-Pivot Quicksort** (fast).
    *   For objects: Uses **MergeSort** (stable, guaranteed `O(N log N)`).
*   `Arrays.binarySearch(arr, key)`: Searches for a key in a sorted array (returns index or negative insertion point). **Array must be sorted first!**
*   `Arrays.fill(arr, val)`: Fills all elements with a specified value.
*   `Arrays.equals(arr1, arr2)`: Checks if two arrays are deeply equal (same elements in same order). For object arrays, `equals()` is used on elements.
*   `Arrays.copyOf(original, newLength)`: Creates a new array and copies elements.
*   `Arrays.copyOfRange(original, from, to)`: Copies a specific range.

```java
import java.util.Arrays;

int[] data = {5, 1, 8, 3};
Arrays.sort(data); // data is now {1, 3, 5, 8}
System.out.println(Arrays.toString(data)); // "[1, 3, 5, 8]"

int index = Arrays.binarySearch(data, 5); // index is 2
```

## 6. Arrays vs. Collections (The "Why" and "When")
*   **Arrays:**
    *   **Fixed Size:** Once created, their size cannot be changed.
    *   **Primitives:** Can directly store primitive types.
    *   **Performance:** Generally faster for low-level memory access and very performance-critical code due to contiguous memory.
*   **Collections (e.g., `ArrayList`):**
    *   **Dynamic Size:** Can grow and shrink as needed.
    *   **Objects Only:** Can only store objects (primitives are auto-boxed).
    *   **Flexibility:** Offer more features (sorting, searching, iteration) and integrate better with generic programming.

**General Advice:** Use `ArrayList` (from the Collections Framework) as your default choice unless you have a specific reason to use raw arrays (e.g., performance-critical code, fixed-size data, native interop).

---

### Links to Topics:
*   [Variables & Primitive Types](01_variables_and_primitive_types.md)
*   [Operators & Expressions](02_operators_and_expressions.md)
*   [Arrays](03_arrays.md)
*   [Type Casting & Conversion](04_type_casting_and_conversion.md)
