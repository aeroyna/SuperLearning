## Custom Sorting

In many interview problems, you don't just sort simple numbers. You often need to sort complex objects or apply a non-standard sorting logic. All modern programming languages provide a way to customize the behavior of their built-in sorting functions, which is a critical and practical skill to master.

### The Core Idea: Defining a Custom Order
The goal of custom sorting is to tell the language's sort function *how* to compare two items. There are two primary ways to do this:

1.  **Using a Key Function (Preferred in Python)**: You provide a function that takes one element and returns a "key". The sorting algorithm then uses these keys to sort the elements. This is simple and powerful. For multi-level sorting, you can return a tuple of keys.
2.  **Using a Comparator Function**: You provide a function that takes two elements, `a` and `b`, and returns a value indicating their relative order (e.g., negative if `a < b`, zero if `a == b`, positive if `a > b`). This is common in languages like Java (`Comparator`) and C++ (`cmp` function).

### Custom Sorting in Python
Python's `sort()` method (for lists) and `sorted()` function are extremely flexible, primarily using the `key` argument.

#### Example 1: Sorting by a Specific Object Attribute
Imagine you have a list of `Student` objects and you want to sort them by `grade`.

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>#include <string>
>#include <iostream>
>using namespace std;
>
>struct Student {
>    string name;
>    int grade;
>
>    Student(string n, int g) : name(n), grade(g) {}
>};
>
>int main() {
>    vector<Student> students = {
>        Student("Alice", 88),
>        Student("Bob", 95),
>        Student("Charlie", 88)
>    };
>
>    // Sort by grade in descending order
>    sort(students.begin(), students.end(),
>         [](const Student& a, const Student& b) {
>             return a.grade > b.grade;  // Descending order
>         });
>
>    // students is now: Bob(95), Alice(88), Charlie(88)
>    for (const auto& s : students) {
>        cout << s.name << ": " << s.grade << endl;
>    }
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>class Student {
>    String name;
>    int grade;
>
>    Student(String name, int grade) {
>        this.name = name;
>        this.grade = grade;
>    }
>
>    @Override
>    public String toString() {
>        return "(" + name + ", " + grade + ")";
>    }
>}
>
>public class CustomSort {
>    public static void main(String[] args) {
>        List<Student> students = Arrays.asList(
>            new Student("Alice", 88),
>            new Student("Bob", 95),
>            new Student("Charlie", 88)
>        );
>
>        // Sort by grade in descending order
>        Collections.sort(students, (s1, s2) -> Integer.compare(s2.grade, s1.grade));
>        // Or use: students.sort(Comparator.comparingInt(s -> -s.grade));
>
>        // students is now: Bob(95), Alice(88), Charlie(88)
>        System.out.println(students);
>    }
>}
>```

>[!example]- Python
>```python
>class Student:
>    def __init__(self, name, grade):
>        self.name = name
>        self.grade = grade
>    def __repr__(self):  # For nice printing
>        return f"({self.name}, {self.grade})"
>
>students = [Student("Alice", 88), Student("Bob", 95), Student("Charlie", 88)]
>
># Sort by grade in descending order.
># The `key` lambda function tells sort to look at the `grade` attribute.
>students.sort(key=lambda student: student.grade, reverse=True)
>
># students is now [("Bob", 95), ("Alice", 88), ("Charlie", 88)]
>print(students)
>```

>[!example]- JavaScript
>```javascript
>class Student {
>    constructor(name, grade) {
>        this.name = name;
>        this.grade = grade;
>    }
>
>    toString() {
>        return `(${this.name}, ${this.grade})`;
>    }
>}
>
>const students = [
>    new Student("Alice", 88),
>    new Student("Bob", 95),
>    new Student("Charlie", 88)
>];
>
>// Sort by grade in descending order
>students.sort((a, b) => b.grade - a.grade);
>
>// students is now: Bob(95), Alice(88), Charlie(88)
>console.log(students.map(s => s.toString()));
>```

#### Example 2: Multi-Level Sorting
What if you want to sort by `grade` (descending), and for ties in grade, sort by `name` (ascending)? You can return a tuple from the key function. Python will sort by the first element of the tuple, then use the second element as a tie-breaker, and so on.

To get descending order for the grade, we can sort by its negative value.

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>#include <string>
>#include <tuple>
>#include <iostream>
>using namespace std;
>
>struct Student {
>    string name;
>    int grade;
>
>    Student(string n, int g) : name(n), grade(g) {}
>};
>
>int main() {
>    vector<Student> students = {
>        Student("Alice", 88),
>        Student("Bob", 95),
>        Student("Charlie", 88)
>    };
>
>    // Sort by grade (desc), then name (asc)
>    sort(students.begin(), students.end(),
>         [](const Student& a, const Student& b) {
>             // First compare by grade (descending)
>             if (a.grade != b.grade)
>                 return a.grade > b.grade;
>             // If grades are equal, compare by name (ascending)
>             return a.name < b.name;
>         });
>
>    // students is now: Bob(95), Alice(88), Charlie(88)
>    // Alice comes before Charlie because of the secondary name sort.
>    for (const auto& s : students) {
>        cout << s.name << ": " << s.grade << endl;
>    }
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>class Student {
>    String name;
>    int grade;
>
>    Student(String name, int grade) {
>        this.name = name;
>        this.grade = grade;
>    }
>
>    @Override
>    public String toString() {
>        return "(" + name + ", " + grade + ")";
>    }
>}
>
>public class MultiLevelSort {
>    public static void main(String[] args) {
>        List<Student> students = Arrays.asList(
>            new Student("Alice", 88),
>            new Student("Bob", 95),
>            new Student("Charlie", 88)
>        );
>
>        // Sort by grade (desc), then by name (asc)
>        Collections.sort(students, (s1, s2) -> {
>            if (s1.grade != s2.grade) {
>                return Integer.compare(s2.grade, s1.grade);  // Descending grade
>            } else {
>                return s1.name.compareTo(s2.name);  // Ascending name
>            }
>        });
>
>        // Alternative using Comparator chaining:
>        // students.sort(Comparator.comparingInt((Student s) -> -s.grade)
>        //                        .thenComparing(s -> s.name));
>
>        // students is now: Bob(95), Alice(88), Charlie(88)
>        // Alice comes before Charlie because of the secondary name sort.
>        System.out.println(students);
>    }
>}
>```

>[!example]- Python
>```python
>class Student:
>    def __init__(self, name, grade):
>        self.name = name
>        self.grade = grade
>    def __repr__(self):
>        return f"({self.name}, {self.grade})"
>
>students = [Student("Alice", 88), Student("Bob", 95), Student("Charlie", 88)]
>
># Sort by grade (desc), then name (asc)
>students.sort(key=lambda s: (-s.grade, s.name))
>
># students is now [("Bob", 95), ("Alice", 88), ("Charlie", 88)]
># Alice comes before Charlie because of the secondary name sort.
>print(students)
>```

>[!example]- JavaScript
>```javascript
>class Student {
>    constructor(name, grade) {
>        this.name = name;
>        this.grade = grade;
>    }
>
>    toString() {
>        return `(${this.name}, ${this.grade})`;
>    }
>}
>
>const students = [
>    new Student("Alice", 88),
>    new Student("Bob", 95),
>    new Student("Charlie", 88)
>];
>
>// Sort by grade (desc), then name (asc)
>students.sort((a, b) => {
>    // First compare by grade (descending)
>    if (a.grade !== b.grade) {
>        return b.grade - a.grade;
>    }
>    // If grades are equal, compare by name (ascending)
>    return a.name.localeCompare(b.name);
>});
>
>// students is now: Bob(95), Alice(88), Charlie(88)
>// Alice comes before Charlie because of the secondary name sort.
>console.log(students.map(s => s.toString()));
>```

### General Pattern for Custom Comparators

Here's a more comprehensive example showing custom comparator patterns in all four languages:

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>#include <string>
>using namespace std;
>
>// Custom comparator as a function
>bool compareStudents(const Student& a, const Student& b) {
>    if (a.grade != b.grade)
>        return a.grade > b.grade;
>    return a.name < b.name;
>}
>
>// Custom comparator as a struct (functor)
>struct StudentComparator {
>    bool operator()(const Student& a, const Student& b) const {
>        if (a.grade != b.grade)
>            return a.grade > b.grade;
>        return a.name < b.name;
>    }
>};
>
>// Usage examples:
>void sortExamples(vector<Student>& students) {
>    // Method 1: Lambda function
>    sort(students.begin(), students.end(),
>         [](const Student& a, const Student& b) {
>             return a.grade > b.grade;
>         });
>
>    // Method 2: Function pointer
>    sort(students.begin(), students.end(), compareStudents);
>
>    // Method 3: Functor
>    sort(students.begin(), students.end(), StudentComparator());
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class CustomComparators {
>    // Method 1: Anonymous class
>    static Comparator<Student> gradeComparator1 = new Comparator<Student>() {
>        @Override
>        public int compare(Student s1, Student s2) {
>            return Integer.compare(s2.grade, s1.grade);
>        }
>    };
>
>    // Method 2: Lambda expression
>    static Comparator<Student> gradeComparator2 =
>        (s1, s2) -> Integer.compare(s2.grade, s1.grade);
>
>    // Method 3: Comparator static methods
>    static Comparator<Student> gradeComparator3 =
>        Comparator.comparingInt((Student s) -> -s.grade);
>
>    // Method 4: Chaining comparators
>    static Comparator<Student> multiLevelComparator =
>        Comparator.comparingInt((Student s) -> -s.grade)
>                  .thenComparing(s -> s.name);
>
>    public static void main(String[] args) {
>        List<Student> students = new ArrayList<>();
>
>        // Usage:
>        Collections.sort(students, gradeComparator1);
>        // Or: students.sort(gradeComparator2);
>    }
>}
>```

>[!example]- Python
>```python
>from functools import cmp_to_key
>
>students = [Student("Alice", 88), Student("Bob", 95), Student("Charlie", 88)]
>
># Method 1: Key function (most Pythonic)
>students.sort(key=lambda s: (-s.grade, s.name))
>
># Method 2: Using operator.attrgetter
>from operator import attrgetter
>students.sort(key=attrgetter('grade'), reverse=True)
>
># Method 3: Custom comparison function (less common)
>def compare_students(s1, s2):
>    if s1.grade != s2.grade:
>        return s2.grade - s1.grade  # Descending
>    if s1.name < s2.name:
>        return -1
>    elif s1.name > s2.name:
>        return 1
>    return 0
>
>students.sort(key=cmp_to_key(compare_students))
>
># Method 4: Using sorted() for a new list
>sorted_students = sorted(students, key=lambda s: (-s.grade, s.name))
>```

>[!example]- JavaScript
>```javascript
>const students = [
>    new Student("Alice", 88),
>    new Student("Bob", 95),
>    new Student("Charlie", 88)
>];
>
>// Method 1: Inline comparator function
>students.sort((a, b) => b.grade - a.grade);
>
>// Method 2: Named comparator function
>function compareByGrade(a, b) {
>    return b.grade - a.grade;
>}
>students.sort(compareByGrade);
>
>// Method 3: Multi-level comparator
>function compareStudents(a, b) {
>    if (a.grade !== b.grade) {
>        return b.grade - a.grade;  // Descending grade
>    }
>    return a.name.localeCompare(b.name);  // Ascending name
>}
>students.sort(compareStudents);
>
>// Method 4: Using arrow function with ternary
>students.sort((a, b) =>
>    a.grade !== b.grade
>        ? b.grade - a.grade
>        : a.name.localeCompare(b.name)
>);
>
>// Note: For creating a new sorted array without modifying original:
>const sortedStudents = [...students].sort((a, b) => b.grade - a.grade);
>```

### Common Custom Sorting Patterns

**Sorting strings by length:**

>[!example]- C++
>```cpp
>vector<string> words = {"apple", "pie", "banana", "cat"};
>sort(words.begin(), words.end(),
>     [](const string& a, const string& b) {
>         return a.length() < b.length();
>     });
>// Result: ["pie", "cat", "apple", "banana"]
>```

>[!example]- Java
>```java
>List<String> words = Arrays.asList("apple", "pie", "banana", "cat");
>words.sort(Comparator.comparingInt(String::length));
>// Result: ["pie", "cat", "apple", "banana"]
>```

>[!example]- Python
>```python
>words = ["apple", "pie", "banana", "cat"]
>words.sort(key=len)
># Result: ["pie", "cat", "apple", "banana"]
>```

>[!example]- JavaScript
>```javascript
>const words = ["apple", "pie", "banana", "cat"];
>words.sort((a, b) => a.length - b.length);
>// Result: ["pie", "cat", "apple", "banana"]
>```

**Sorting by absolute value:**

>[!example]- C++
>```cpp
>#include <cmath>
>vector<int> numbers = {-5, 3, -8, 1, 7};
>sort(numbers.begin(), numbers.end(),
>     [](int a, int b) {
>         return abs(a) < abs(b);
>     });
>// Result: [1, 3, -5, 7, -8]
>```

>[!example]- Java
>```java
>List<Integer> numbers = Arrays.asList(-5, 3, -8, 1, 7);
>numbers.sort(Comparator.comparingInt(Math::abs));
>// Result: [1, 3, -5, 7, -8]
>```

>[!example]- Python
>```python
>numbers = [-5, 3, -8, 1, 7]
>numbers.sort(key=abs)
># Result: [1, 3, -5, 7, -8]
>```

>[!example]- JavaScript
>```javascript
>const numbers = [-5, 3, -8, 1, 7];
>numbers.sort((a, b) => Math.abs(a) - Math.abs(b));
>// Result: [1, 3, -5, 7, -8]
>```

Mastering custom sorting is essential. Problems like "Largest Number" (LeetCode #179) or sorting log files are classic examples where a custom comparator is the key to the entire solution.
