## String Manipulation

Strings are one of the most common data types encountered in coding interviews. Understanding how to manipulate them efficiently is crucial. A key concept to grasp is the difference between mutable and immutable strings, as it heavily influences the performance of your code.

### Immutable vs. Mutable Strings

- **Immutable Strings**: In many popular languages, including Python and Java, strings are immutable. This means that once a string is created, it cannot be changed. Any operation that appears to modify a string actually creates a *new* string in memory.

  - **Pros**: Safety. Immutable strings are thread-safe and can be used as keys in hash maps because they are guaranteed not to change.
  - **Cons**: Performance. Performing many modifications (like concatenating in a loop) can be very inefficient as it involves creating many intermediate string objects.

- **Mutable Strings**: In languages like C++, strings are mutable. You can change individual characters without creating a new string object.

### The Problem with Loop Concatenation

A common mistake is building a string in a loop using concatenation:

>[!example]- C++
>```cpp
>// In C++, strings are mutable but repeated concatenation
>// still involves reallocation as string grows
>// O(n^2) in worst case without capacity pre-allocation
>string result = "";
>for (char c : my_character_list) {
>    result += c;  // May reallocate and copy
>}
>```

>[!example]- Java
>```java
>// Inefficient O(n^2) approach in Java
>// String is immutable - creates new object each time
>String result = "";
>for (char c : myCharacterList) {
>    result += c;  // Creates a new String object each iteration
>}
>```

>[!example]- Python
>```python
># Inefficient O(n^2) approach
>result = ""
>for char in my_character_list:
>    result += char # This creates a new string in every iteration
>```

>[!example]- JavaScript
>```javascript
>// Inefficient O(n^2) approach in JavaScript
>// Strings are immutable - creates new string each time
>let result = "";
>for (const char of myCharacterList) {
>    result += char;  // Creates a new string each iteration
>}
>```

Because a new string is created in each iteration, the total time complexity becomes O(n^2).

### Efficient String Building: The Builder Pattern

To efficiently build strings from multiple pieces, you should use a "string builder" pattern. In Python, the idiomatic way to do this is to append all the pieces to a list and then use the `.join()` method at the end.

>[!example]- C++
>```cpp
>// Efficient O(n) approach using vector
>// Pre-allocate if size is known for even better performance
>vector<char> parts;
>for (char c : my_character_list) {
>    parts.push_back(c);
>}
>
>// Create final string efficiently
>string result(parts.begin(), parts.end());
>
>// Alternative: use string and reserve capacity
>string result2;
>result2.reserve(my_character_list.size());
>for (char c : my_character_list) {
>    result2 += c;
>}
>```

>[!example]- Java
>```java
>// Efficient O(n) approach using StringBuilder
>StringBuilder parts = new StringBuilder();
>for (char c : myCharacterList) {
>    parts.append(c);
>}
>
>// Convert to string in one efficient operation
>String result = parts.toString();
>```

>[!example]- Python
>```python
># Efficient O(n) approach
>parts = []
>for char in my_character_list:
>    parts.append(char)
>
># .join() creates the final string in one efficient operation
>result = "".join(parts)
>```

>[!example]- JavaScript
>```javascript
>// Efficient O(n) approach using array
>const parts = [];
>for (const char of myCharacterList) {
>    parts.push(char);
>}
>
>// join() creates the final string in one efficient operation
>const result = parts.join("");
>```

This approach ensures that the string is built in linear time, O(n), which is significantly faster for large strings.

### Key String Operations

- **Substrings/Slicing**: Extracting a portion of a string. This is typically an efficient operation.
- **Searching (`find`, `index`)**: Finding the index of a character or substring.
- **Replacing**: Creating a new string with parts replaced.
- **Splitting and Joining**: Breaking a string into a list of substrings based on a delimiter, and the reverse operation.
- **Type Conversion**: Converting strings to integers and vice-versa.
