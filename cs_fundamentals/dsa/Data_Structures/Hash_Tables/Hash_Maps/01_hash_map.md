## Hash Map

A hash map, often called a dictionary, is a data structure that stores data as **key-value pairs**. It uses a hash function to map a key to a location in memory where the corresponding value is stored. This design allows for highly efficient lookups, insertions, and deletions.

### Core Idea
The fundamental principle of a hash map is to provide, on average, O(1) time complexity for `get`, `set`, and `delete` operations. When you provide a key, the hash map computes its hash to instantly find the "bucket" where the value is located, avoiding the need to search through all the elements.

### Use Cases
Hash maps are arguably one of the most useful data structures in programming and are a go-to tool for solving a vast range of interview problems.
- **Efficient Lookups**: The classic use case. For example, in the "Two Sum" problem, you can store numbers you've seen and their indices in a hash map to find a required complement in O(1) time.
- **Counting/Frequency Analysis**: A hash map is perfect for counting the occurrences of items, such as characters in a string or numbers in an array. The item is the key, and the count is the value.
- **Caching/Memoization**: Storing the results of expensive computations to avoid recalculating them. The function inputs can be a key, and the result is the value.

### Implementation Examples

>[!example]- C++
>```cpp
>#include <unordered_map>
>#include <string>
>#include <iostream>
>
>int main() {
>    // 1. Initialize a hash map
>    std::unordered_map<std::string, int> student_ages;
>    // or with initial values
>    student_ages = {{"Alice", 24}, {"Bob", 25}};
>
>    // 2. Insert or update a key-value pair (O(1) on average)
>    student_ages["Charlie"] = 26; // Insert
>    student_ages["Alice"] = 25;   // Update
>
>    // 3. Access a value by key
>    if (student_ages.find("Alice") != student_ages.end()) {
>        int alice_age = student_ages["Alice"]; // Returns 25
>    }
>
>    // A safer way to access is with .at(), which throws an exception if key not found
>    try {
>        int bob_age = student_ages.at("Bob"); // Returns 25
>    } catch (const std::out_of_range& e) {
>        std::cout << "Key not found." << std::endl;
>    }
>
>    // 4. Check for existence of a key
>    if (student_ages.find("Charlie") != student_ages.end()) {
>        std::cout << "Charlie is in the map." << std::endl;
>    }
>
>    // 5. Delete a key-value pair
>    student_ages.erase("Bob");
>
>    // 6. Iterate over the map
>    // Iterate over key-value pairs
>    for (const auto& [name, age] : student_ages) {
>        std::cout << name << " is " << age << " years old." << std::endl;
>    }
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>import java.util.Map;
>
>public class HashMapExample {
>    public static void main(String[] args) {
>        // 1. Initialize a hash map
>        Map<String, Integer> studentAges = new HashMap<>();
>        // or with initial values
>        studentAges = new HashMap<>(Map.of("Alice", 24, "Bob", 25));
>
>        // 2. Insert or update a key-value pair (O(1) on average)
>        studentAges.put("Charlie", 26); // Insert
>        studentAges.put("Alice", 25);   // Update
>
>        // 3. Access a value by key
>        if (studentAges.containsKey("Alice")) {
>            int aliceAge = studentAges.get("Alice"); // Returns 25
>        }
>
>        // A safer way to access is with .getOrDefault()
>        int bobAge = studentAges.getOrDefault("Bob", -1); // Returns 25
>        int danAge = studentAges.getOrDefault("Dan", -1); // Returns -1
>
>        // 4. Check for existence of a key
>        if (studentAges.containsKey("Charlie")) {
>            System.out.println("Charlie is in the map.");
>        }
>
>        // 5. Delete a key-value pair
>        studentAges.remove("Bob");
>
>        // 6. Iterate over the map
>        // Iterate over keys
>        for (String name : studentAges.keySet()) {
>            System.out.println(name + " " + studentAges.get(name));
>        }
>
>        // Iterate over key-value pairs
>        for (Map.Entry<String, Integer> entry : studentAges.entrySet()) {
>            System.out.println(entry.getKey() + " is " + entry.getValue() + " years old.");
>        }
>    }
>}
>```

>[!example]- Python
>```python
># 1. Initialize a hash map
>student_ages = {}
># or with initial values
>student_ages = {"Alice": 24, "Bob": 25}
>
># 2. Insert or update a key-value pair (O(1) on average)
>student_ages["Charlie"] = 26 # Insert
>student_ages["Alice"] = 25   # Update
>
># 3. Access a value by key
>try:
>    alice_age = student_ages["Alice"] # Returns 25
>except KeyError:
>    print("Key not found.")
>
># A safer way to access is with .get(), which returns None if the key is not found
>bob_age = student_ages.get("Bob") # Returns 25
>dan_age = student_ages.get("Dan") # Returns None
>
># 4. Check for existence of a key
>if "Charlie" in student_ages:
>    print("Charlie is in the map.")
>
># 5. Delete a key-value pair
>del student_ages["Bob"]
>
># 6. Iterate over the map
># Iterate over keys
>for name in student_ages:
>    print(name, student_ages[name])
>
># Iterate over key-value pairs
>for name, age in student_ages.items():
>    print(f"{name} is {age} years old.")
>```

>[!example]- JavaScript
>```javascript
>// 1. Initialize a hash map
>let studentAges = {};
>// or with initial values
>studentAges = {"Alice": 24, "Bob": 25};
>
>// Alternative: Using Map object (recommended for better performance)
>let studentAgesMap = new Map();
>studentAgesMap = new Map([["Alice", 24], ["Bob", 25]]);
>
>// 2. Insert or update a key-value pair (O(1) on average)
>// Using object
>studentAges["Charlie"] = 26; // Insert
>studentAges["Alice"] = 25;   // Update
>
>// Using Map
>studentAgesMap.set("Charlie", 26);
>studentAgesMap.set("Alice", 25);
>
>// 3. Access a value by key
>// Using object
>if ("Alice" in studentAges) {
>    let aliceAge = studentAges["Alice"]; // Returns 25
>}
>
>// Using Map
>if (studentAgesMap.has("Alice")) {
>    let aliceAge = studentAgesMap.get("Alice"); // Returns 25
>}
>
>// A safer way with default values
>let bobAge = studentAges["Bob"] || null; // Returns 25
>let danAge = studentAges["Dan"] || null; // Returns null
>
>// 4. Check for existence of a key
>if ("Charlie" in studentAges) {
>    console.log("Charlie is in the map.");
>}
>
>// 5. Delete a key-value pair
>delete studentAges["Bob"];
>// Using Map
>studentAgesMap.delete("Bob");
>
>// 6. Iterate over the map
>// Iterate over keys (object)
>for (let name in studentAges) {
>    console.log(name, studentAges[name]);
>}
>
>// Iterate over key-value pairs (Map)
>for (let [name, age] of studentAgesMap) {
>    console.log(`${name} is ${age} years old.`);
>}
>```
